<!--
title: OCI Bucket Navigator - List, Index and Download Object Storage files from the terminal (oci_download bash function)
description: A reusable bash function for Oracle Cloud Infrastructure (OCI) Object Storage that lists bucket objects as a numbered table and downloads any of them by index, using the OCI CLI and jq. Install it in your .bashrc.
keywords: OCI Object Storage download CLI, oci os object get, oci os object list, download bucket object by index, OCI CLI bash function, jq awk human readable size, .bashrc oci_download, Oracle Cloud bucket terminal, namespace bucket-name, OCI_CLI_PROFILE, list objects in OCI bucket
author: charles-josiah
-->

# ☁️ OCI Bucket Navigator — Guide & the `oci_download` Function

This guide documents a *shell* function to **list, index and download** objects from an **Oracle Cloud Infrastructure (OCI)** bucket straight from the terminal, picking the file by the **index** shown in the listing.

**Keywords for search:** download OCI Object Storage file from CLI · `oci os object get` by index · list bucket objects as a numbered table · OCI CLI + jq bash helper · install `oci_download` in `.bashrc`

> **Why put it in `.bashrc`?**
> By adding the `exports` and the `oci_download` function to your `~/.bashrc` (or `~/.bash_profile`), they become **available in every session** of your user without copy/pasting each time. Just open a new terminal (or run `source ~/.bashrc`) and use the command.

---

## ✅ Prerequisites

- **OCI CLI** installed and authenticated (`oci setup config`)
- **jq** to handle JSON
- Permissions on the bucket (proper policy in the tenancy/compartment)
- Network access to the Object Storage endpoint

---

## 🔧 Environment variables

Add to your `~/.bashrc` (adjusting the values):

```bash
# Silences warnings from old Python/OCI CLI libs
export PYTHONWARNINGS="ignore"

# Bucket identification in OCI
export BUCKET_NAME="YOUR_BUCKET"
export NAMESPACE="YOUR_NAMESPACE"
```

> Tip: if you work with multiple environments, consider also exporting `OCI_CLI_PROFILE` (e.g. `dev`, `prod`) to switch between profiles in `~/.oci/config`.

After editing `.bashrc`, reload it:
```bash
source ~/.bashrc
```

---

## 🧩 The `oci_download` function (index-based)

Paste the function below into the **same `.bashrc`** (or into a functions file and `source` it):

```bash
oci_download() {

  local idx="${1:-}"
  local dest="${2:-/tmp}"
  local obj_name out

  echo "=== Object list in bucket $BUCKET_NAME ==="
  oci os object list \
    --namespace-name "$NAMESPACE" \
    --bucket-name "$BUCKET_NAME" \
    --all \
  | jq -r '.data | to_entries[] | [ .key, .value.size, .value."time-created", .value.name ] | @tsv' \
  | awk -F'\t' '
    function human(n, units, i) {
      split("B KB MB GB TB PB", units, " ");
      for (i=1; n>=1024 && i<length(units); i++) n/=1024;
      return sprintf("%.2f %s", n, units[i]);
    }
    { printf "%5d | %-10s | %-23s | %s\n", $1, human($2), $3, $4 }
  '

  echo
  read -rp "Enter the index of the object to download, or q|Q to quit: " idx
  if [[ "$idx" == "q" || "$idx" == "Q" ]]; then
    echo "Exiting without downloading anything..."
    return 0
  fi

  obj_name="$(
    oci os object list \
      --namespace-name "$NAMESPACE" \
      --bucket-name "$BUCKET_NAME" \
      --all \
    | jq -r --argjson i "$idx" '.data[$i].name'
  )"

  if [[ -z "$obj_name" || "$obj_name" == "null" ]]; then
    echo "Invalid index!"
    return 1
  fi

  mkdir -p "$dest"
  out="$dest/$(basename -- "$obj_name")"
  echo "Downloading: $obj_name -> $out"

  oci os object get \
    --namespace-name "$NAMESPACE" \
    --bucket-name "$BUCKET_NAME" \
    --name "$obj_name" \
    --file "$out"
}
```

**What does it do?**
1. Calls `oci os object list` and turns the JSON into a **numbered table** (via `jq` + `awk`), where the first column is the **index** (the entry's `.key`).
2. Asks the user for an **index**; `q`/`Q` exits without downloading.
3. Resolves the **object name** by position `.data[$idx].name`.
4. Downloads it with `oci os object get` into `dest` (default: `/tmp`).

---

## ▶️ Usage examples

List and download (interactive mode):
```bash
oci_download
# ... the function lists objects and asks: "Enter the index..."
```

Passing a destination folder (still prompts for the index):
```bash
oci_download "" ~/Downloads
```

Download and save into `/var/tmp` (with prompt):
```bash
oci_download "" /var/tmp
```

> The function is **interactive** by design: even with parameters, it always asks for the index to avoid mistakes when downloading among many objects.

---

## 🧪 Sample output (listing)

```
    0 |  12.54 MB | 2025-09-15T10:12:34.000Z | backups/db-snapshot-2025-09-15.sql.gz
    1 | 803.00 KB | 2025-09-14T22:01:10.000Z | logs/app-2025-09-14.log.gz
    2 |   2.10 GB | 2025-09-14T01:22:00.000Z | images/disk.qcow2
```

---

## 🧠 Why in `.bashrc`?

- **Always available**: you gain a new shell "command" to browse/download from the bucket.
- **Standardizes the environment**: the `BUCKET_NAME`/`NAMESPACE` variables stay defined for all terminals.
- **Less friction**: avoids repeating exports/definitions and reduces operational errors.

> To install it for **all users**, place a file in `/etc/profile.d/oci_download.sh` with the `exports` and the function, and set appropriate permissions.

---

## 🛡️ Best practices & notes

- **Profiles**: use `export OCI_CLI_PROFILE=prod` to lock the current profile.
- **Quotas/latency**: `--all` paginates everything; on huge buckets it can be slow. If needed, filter with `--prefix` and/or switch to manual pagination.
- **Security**: do not version-control `~/.oci/config` with keys; use **Vault**/KMS or controlled variables.
- **Common errors**:
  - `NotAuthorizedOrNotFound`: check the user/dynamic-group policy in the compartment.
  - `jq: command not found`: install `jq`.
  - `ObjectNotFound`: the object may have been removed between the listing and the *get* (concurrency).

---

## 🔁 Useful variations (optional)

- **Filter by prefix** (e.g. only `logs/`):
  ```bash
  oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET_NAME" --prefix "logs/"
  ```
- **Download without a prompt** (when you already know the exact name):
  ```bash
  oci os object get --namespace-name "$NAMESPACE" --bucket-name "$BUCKET_NAME" --name "path/file.ext" --file "/tmp/file.ext"
  ```

---

## 🧷 Quick install checklist

1. Install `oci` and `jq`.
2. Configure `oci setup config`.
3. Edit `~/.bashrc`: add the `exports` and the `oci_download` function.
4. `source ~/.bashrc`.
5. Run `oci_download` and pick the index.

---

> 🇧🇷 **Versão em português:** [Linux-OCI-Bucket_Navigator.md](Linux-OCI-Bucket_Navigator.md)
