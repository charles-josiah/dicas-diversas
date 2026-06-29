<!--
title: Persistent Secondary Private IPs on OCI (Oracle Linux) with NetworkManager Dispatcher
description: How to make OCI VNIC secondary private IP addresses persist across reboots on Oracle Linux using a NetworkManager dispatcher script. Fixes secondary IP disappearing after reboot.
keywords: OCI secondary IP, Oracle Cloud Infrastructure, Oracle Linux, NetworkManager dispatcher, secondaryPrivateIps, VNIC secondary IP not persistent, secondary IP disappears after reboot, ip addr replace, dhcp4-change, ens3, instance metadata service, IMDS, 169.254.169.254, policy based routing, oci-network-config, nmcli, SELinux restorecon
author: charles-josiah
-->

# Persistent Secondary Private IPs on OCI (Oracle Linux) via NetworkManager Dispatcher

> **TL;DR** — On Oracle Cloud Infrastructure (OCI) VMs running Oracle Linux, the Oracle DHCP only hands the **primary** VNIC IP to the operating system. Secondary private IPs assigned in the console show up in OCI but **do not come up inside the OS**. Worse, the OCI environment **recreates the NetworkManager profile on every boot** (a fresh `Wired Connection`, new UUID, pure DHCP), discarding any static address saved in a profile. The robust fix is a **NetworkManager dispatcher script** that re-applies the secondary IPs whenever the interface comes up or DHCP renews.

**Keywords for search:** OCI secondary IP not persistent · secondary private IP disappears after reboot · Oracle Linux VNIC multiple IPs · NetworkManager dispatcher OCI · `secondaryPrivateIps` · `ip addr replace` · instance metadata `/opc/v2/vnics/`

---

## 0. Identify your scenario first

Oracle's documentation separates two cases that need different handling. **Identify yours before proceeding.**

| Scenario | What it is | Approach |
|---|---|---|
| **A — Secondary IPs on the primary VNIC** | Several private IPs (`secondaryPrivateIps`) on the same VNIC as the primary IP. Most common case, and the one this guide covers. | NetworkManager dispatcher (this document). |
| **B — Secondary VNIC** | A second virtual NIC attached to the instance, with its own IP. | `oci-network-config` (official Oracle utility) **or** policy-based routing. See section 8. |

> **Oracle's official recommendation:** to avoid asymmetric routing on Linux, Oracle recommends **assigning multiple private IPs to a single VNIC** rather than using multiple VNICs from the same CIDR. In other words, scenario A (this guide) is the approach Oracle itself advises when you need multiple IPs on the same subnet.

---

## 1. Discovery

### 1.1. Interface name

```bash
ip -br addr show
nmcli device status
```

Note the interface holding the primary IP (typically `ens3`). **Examples use `ens3` — substitute your real name.**

### 1.2. Find the secondary IPs

**Option A — console:** Instance → VNIC → *IP administration* → *IPv4 addresses*.

**Option B — instance metadata (IMDS), more reliable:** OCI exposes the VNIC configuration, including secondary IPs, via the metadata service. This avoids typos:

```bash
curl -s -H "Authorization: Bearer Oracle" http://169.254.169.254/opc/v2/vnics/ | python3 -m json.tool
```

Look for the `secondaryPrivateIps` array. Also note `subnetCidrBlock` (e.g. `192.168.123.0/24`) — that is where the **prefix** for the OS comes from.

> **Prefix caveat:** the console shows IPs as `/32` on the edit screen. That is only the OCI object representation. **Inside the OS, use the subnet prefix** (usually `/24`) so the host treats the addresses as on-link and needs no extra route.

---

## 2. Remove competing configuration

If you created static NetworkManager profiles for these IPs in earlier attempts, **disable them** — two "owners" of the same IP cause unpredictable boot behavior.

```bash
# list all profiles
nmcli -t -f NAME,UUID,DEVICE,ACTIVE,AUTOCONNECT,FILENAME connection show

# disable autoconnect on a custom profile (reference by UUID if names are duplicated)
sudo nmcli connection modify <UUID-or-name> connection.autoconnect no

# or delete it entirely
sudo nmcli connection delete <UUID-or-name>
```

> **Do not touch** profiles for other interfaces (`docker0`, `enp1s0`, etc.). And **do not try to delete** the `Wired Connection` that OCI generates at boot — it is recreated on every reboot. The strategy is to let it handle the **primary** IP via DHCP, while the dispatcher handles the **secondary** ones.

---

## 3. Create the dispatcher script

The script goes in `/etc/NetworkManager/dispatcher.d/`. The numeric prefix sets the run order.

### 3.1. Fixed IP list version (simplest)

The direct approach. Edit the `IPS` list for your server:

```bash
sudo tee /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips >/dev/null <<'EOF'
#!/bin/bash
# Re-applies secondary IPs of the primary VNIC (OCI / Oracle Linux)
# Fires on the up / dhcp4-change events of the target interface.

IFACE="$1"
STATUS="$2"
DEV="ens3"

IPS="
192.168.123.82/24
192.168.123.141/24
192.168.123.235/24
192.168.123.60/24
192.168.123.206/24
"

IPBIN="/usr/sbin/ip"
[ -x "$IPBIN" ] || IPBIN="/sbin/ip"

case "$STATUS" in
  up|dhcp4-change|connectivity-change)
    if [ "$IFACE" = "$DEV" ]; then
      for IPADDR in $IPS; do
        # addr replace is idempotent: does not fail if the IP already exists
        "$IPBIN" addr replace "$IPADDR" dev "$DEV"
      done
    fi
    ;;
esac

exit 0
EOF
```

### 3.2. Self-configuring version (reads IPs from metadata)

This version **needs no maintenance** when you add/remove IPs in the console — it reads the list straight from the instance metadata on each run. More robust for server fleets.

```bash
sudo tee /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips >/dev/null <<'EOF'
#!/bin/bash
# Re-applies secondary IPs by reading the primary VNIC from OCI metadata (IMDS v2).

IFACE="$1"
STATUS="$2"
DEV="ens3"

IPBIN="/usr/sbin/ip"
[ -x "$IPBIN" ] || IPBIN="/sbin/ip"

case "$STATUS" in
  up|dhcp4-change|connectivity-change)
    if [ "$IFACE" = "$DEV" ]; then
      # subnet prefix (e.g. 24) from subnetCidrBlock
      PREFIX=$(curl -s -H "Authorization: Bearer Oracle" \
        http://169.254.169.254/opc/v2/vnics/ \
        | python3 -c 'import sys,json; print(json.load(sys.stdin)[0]["subnetCidrBlock"].split("/")[1])' 2>/dev/null)
      [ -z "$PREFIX" ] && PREFIX=24

      # list of secondary IPs
      IPS=$(curl -s -H "Authorization: Bearer Oracle" \
        http://169.254.169.254/opc/v2/vnics/ \
        | python3 -c 'import sys,json; [print(ip) for ip in json.load(sys.stdin)[0].get("secondaryPrivateIps",[])]' 2>/dev/null)

      for IP in $IPS; do
        "$IPBIN" addr replace "${IP}/${PREFIX}" dev "$DEV"
      done
    fi
    ;;
esac

exit 0
EOF
```

> Use **3.1** if you prefer explicit control and predictability; use **3.2** if you want the server to track whatever is assigned on the VNIC automatically. Both use the same file name — pick one.

---

## 4. Permissions and SELinux

NetworkManager only runs dispatcher scripts that are `root:root`, not group/other-writable, and executable. On Oracle Linux (SELinux enforcing) the file context matters too.

```bash
sudo chown root:root /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo chmod 755 /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo restorecon -v /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips 2>/dev/null || true

ls -l /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
```

Expected: `-rwxr-xr-x. root root` (the `.` indicates SELinux context applied).

---

## 5. Why these choices (technical rationale)

| Decision | Reason |
|---|---|
| **Dispatcher** instead of `rc.local` | `rc.local` runs **once at boot**. The dispatcher fires on `up` and `dhcp4-change`, also covering interface recycling at runtime (DHCP renewal, `nmcli device reapply`). |
| **`ip addr replace`** instead of `ip addr add` | `replace` is idempotent: it neither fails nor floods the log if the IP already exists. `add` returns an error when the address is already present. (Also avoids the `grep -w` false positive, where searching `192.168.123.6` would match inside `192.168.123.60`.) |
| **`$IFACE = $DEV` filter** | The dispatcher fires for every interface (including `docker0`); the filter ensures we only act on the target interface. |
| **`restorecon`** | Without the correct SELinux context, NM may be silently prevented from running the script. |
| **IPs on the primary VNIC, not multiple VNICs** | Oracle's recommendation to avoid asymmetric routing on Linux. |

---

## 6. Testing

### 6.1. Runtime test (interface recycle) — validates the mechanism

```bash
sudo nmcli device reapply ens3
ip -4 addr show ens3
```

> Running the script "by hand" (`sudo .../99-oci-secondary-ips ens3 up`) only tests the script **logic**, not whether NetworkManager invokes it. Prefer `device reapply` or a reboot.

### 6.2. Check all IPs are present

```bash
for ip in 192.168.123.82 192.168.123.141 192.168.123.235 192.168.123.60 192.168.123.206; do
  ip -4 addr show dev ens3 | grep -qw "$ip" && echo "$ip OK on ens3" || echo "$ip MISSING"
done
```

### 6.3. Check reachability over the network

> **Important:** pinging the IPs **from the same machine** always succeeds, because traffic never leaves the interface — this only confirms the IP is *assigned*, not that it is *reachable*. To validate real reachability, ping **from another host on the subnet**:
>
> ```bash
> # run from ANOTHER host on the same subnet
> for ip in 192.168.123.82 192.168.123.141 192.168.123.235 192.168.123.60 192.168.123.206; do
>   ping -c2 -W1 "$ip" >/dev/null && echo "$ip OK" || echo "$ip FAIL"
> done
> ```

### 6.4. Definitive test — reboot

```bash
sudo reboot
```

After reconnecting:

```bash
ip -4 addr show ens3
nmcli connection show --active
```

`nmcli` will likely show an OCI-recreated `Wired Connection` active on the interface — **that is expected**. What matters is that the secondary IPs appear in `ip addr` with no manual intervention.

> **Access caution:** if your SSH arrives over this interface, keep the OCI serial/VNC console handy before rebooting. The primary IP returns within seconds, but it is your safety net.

---

## 7. Maintenance (add/remove IPs)

Edit the list (version 3.1 only; 3.2 updates itself):

```bash
sudo vi /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo nmcli device reapply ens3
ip -4 addr show ens3
```

Remove an IP still active on the interface (the dispatcher does not remove it automatically when you drop it from the list):

```bash
sudo ip addr del 192.168.123.82/24 dev ens3
```

---

## 8. Scenario B — Secondary VNIC (not covered by the flow above)

If the IPs are on a **secondary VNIC** (an extra NIC attached to the instance), the handling changes:

- **Official Oracle path:** use the `oci-network-config` utility (part of the `oci-utils` package), recommended by Oracle for configuring secondary VNICs on Oracle Linux. On RHEL 9.6+/10.0+ there is also `nm-cloud-setup` with `NM_CLOUD_SETUP_OCI=yes`.
- **Policy-based routing:** a secondary VNIC needs a dedicated route table and source rules, otherwise return traffic exits via the wrong interface (asymmetric routing). Pattern: `ip rule add from <source-ip> lookup <table>` + a default route in the table pointing to the VNIC's `virtualRouterIp`.
- **iSCSI caution:** adding VNICs can divert iSCSI traffic away from the primary VNIC and break attached volumes. iSCSI boot volumes use `169.254.0.2/32` and block volumes use `169.254.2.0/24`; if needed, add specific routes for those destinations via the primary VNIC's router.

For that scenario, follow the official documentation (`oci-network-config`) instead of the manual dispatcher.

---

## 9. References

- **Virtual Network Interface Cards (VNICs)** — overview of VNICs, secondary IPs, instance metadata (IMDS) and OS configuration for secondary VNICs:
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm
- **Configuring Linux to Use a Secondary Private IP Address** — official guidance on secondary IP persistence on Linux (including the case where NetworkManager overwrites the configuration at boot):
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIPaddresses_topic-Linux_Details_about_Secondary_IP_Addresses.htm
- **IP Addresses (Private IP Objects)** — private IP assignment behavior and secondary objects on the VNIC:
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIPaddresses.htm
- **Instance Metadata Service (IMDS)** — `/opc/v2/vnics/` endpoints used by the self-configuring script version:
  https://docs.oracle.com/iaas/Content/Compute/Tasks/gettingmetadata.htm
- **oci-network-config (oci-utils)** — official Oracle utility for configuring secondary VNICs on Oracle Linux (scenario B):
  https://docs.oracle.com/iaas/oracle-linux/oci-utils/index.htm

---

> 🇧🇷 **Versão em português:** [Linux-OCI-IPs_Secundarios_NetworkManager.md](Linux-OCI-IPs_Secundarios_NetworkManager.md)
