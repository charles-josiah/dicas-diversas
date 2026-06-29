# HOWTO — IPs Secundários Persistentes em VM OCI (Oracle Linux) via NetworkManager Dispatcher

> **Resumo:** Em VMs do Oracle Cloud Infrastructure (OCI) com Oracle Linux, o DHCP da Oracle entrega ao sistema operacional **apenas o IP primário** da VNIC. IPs privados secundários atribuídos pelo console aparecem na OCI, mas **não sobem sozinhos** dentro do SO. Além disso, o ambiente OCI **recria o perfil de rede do NetworkManager a cada boot** (um `Wired Connection` novo, com UUID diferente, em DHCP puro), descartando endereços estáticos gravados em perfis. A solução robusta é um **script dispatcher do NetworkManager** que reaplica os IPs secundários sempre que a interface sobe ou o DHCP renegocia.

---

## 0. Antes de começar: qual é o seu cenário?

A documentação da Oracle separa dois casos que exigem tratamentos diferentes. **Identifique o seu antes de seguir.**

| Cenário | O que é | Abordagem |
|---|---|---|
| **A — IPs secundários na VNIC primária** | Vários IPs privados (`secondaryPrivateIps`) na mesma VNIC do IP principal. É o caso mais comum e o coberto por este HOWTO. | Dispatcher do NetworkManager (este documento). |
| **B — VNIC secundária** | Uma segunda placa de rede virtual anexada à instância, com IP próprio. | `oci-network-config` (utilitário oficial Oracle) **ou** policy-based routing. Veja a seção 8. |

> **Recomendação oficial da Oracle:** para evitar roteamento assimétrico no Linux, a Oracle recomenda **atribuir vários IPs privados a uma única VNIC** em vez de usar várias VNICs do mesmo CIDR. Ou seja, o cenário A (este HOWTO) é a abordagem que a própria Oracle aconselha quando você precisa de múltiplos IPs na mesma subnet.

---

## 1. Levantamento

### 1.1. Nome da interface

```bash
ip -br addr show
nmcli device status
```

Anote a interface com o IP primário (tipicamente `ens3`). **Os exemplos usam `ens3` — substitua pelo nome real.**

### 1.2. Descobrir os IPs secundários

**Opção A — pelo console:** Instância → VNIC → *IP administration* → *IPv4 addresses*.

**Opção B — pelo metadata da instância (IMDS), mais confiável:** a OCI expõe a configuração da VNIC, incluindo os IPs secundários, no metadata service. Isso evita erro de digitação:

```bash
curl -s -H "Authorization: Bearer Oracle" http://169.254.169.254/opc/v2/vnics/ | python3 -m json.tool
```

Procure pelo array `secondaryPrivateIps`. Anote também o `subnetCidrBlock` (ex.: `192.168.123.0/24`) — é dele que sai o **prefixo** a usar no SO.

> **Atenção ao prefixo:** o console mostra os IPs como `/32` na tela de edição. Isso é apenas a representação do objeto na OCI. **Dentro do SO use o prefixo da subnet** (normalmente `/24`), para que o host trate os endereços como on-link e não precise de rota extra.

---

## 2. Remover configurações concorrentes

Se você criou perfis estáticos no NetworkManager para esses IPs em tentativas anteriores, **desative-os** — dois "donos" do mesmo IP causam comportamento imprevisível no boot.

```bash
# listar todos os perfis
nmcli -t -f NAME,UUID,DEVICE,ACTIVE,AUTOCONNECT,FILENAME connection show

# desativar autoconnect de um perfil customizado (use UUID se houver nomes duplicados)
sudo nmcli connection modify <UUID-ou-nome> connection.autoconnect no

# ou remover de vez
sudo nmcli connection delete <UUID-ou-nome>
```

> **Não mexa** em perfis de outras interfaces (`docker0`, `enp1s0`, etc.). E **não tente apagar** o `Wired Connection` que o OCI gera no boot — ele é recriado a cada reinicialização. A estratégia é deixá-lo cuidar do IP **primário** via DHCP, enquanto o dispatcher cuida dos **secundários**.

---

## 3. Criar o script dispatcher

O script vai em `/etc/NetworkManager/dispatcher.d/`. O prefixo numérico define a ordem.

### 3.1. Versão com lista fixa de IPs (mais simples)

Esta é a abordagem direta. Edite a lista `IPS` conforme o seu servidor:

```bash
sudo tee /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips >/dev/null <<'EOF'
#!/bin/bash
# Reaplica IPs secundarios da VNIC primaria (OCI / Oracle Linux)
# Dispara nos eventos up / dhcp4-change da interface alvo.

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
        # addr replace e idempotente: nao falha se o IP ja existir
        "$IPBIN" addr replace "$IPADDR" dev "$DEV"
      done
    fi
    ;;
esac

exit 0
EOF
```

### 3.2. Versão auto-configurável (lê os IPs do metadata)

Esta versão **não precisa de manutenção** quando você adiciona/remove IPs no console — ela lê a lista direto do metadata da instância a cada execução. Mais robusta para frotas de servidores.

```bash
sudo tee /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips >/dev/null <<'EOF'
#!/bin/bash
# Reaplica IPs secundarios lendo a VNIC primaria do metadata OCI (IMDS v2).

IFACE="$1"
STATUS="$2"
DEV="ens3"

IPBIN="/usr/sbin/ip"
[ -x "$IPBIN" ] || IPBIN="/sbin/ip"

case "$STATUS" in
  up|dhcp4-change|connectivity-change)
    if [ "$IFACE" = "$DEV" ]; then
      # prefixo da subnet (ex.: 24) a partir do subnetCidrBlock
      PREFIX=$(curl -s -H "Authorization: Bearer Oracle" \
        http://169.254.169.254/opc/v2/vnics/ \
        | python3 -c 'import sys,json; print(json.load(sys.stdin)[0]["subnetCidrBlock"].split("/")[1])' 2>/dev/null)
      [ -z "$PREFIX" ] && PREFIX=24

      # lista de IPs secundarios
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

> Use a **3.1** se preferir controle explícito e previsibilidade; use a **3.2** se quiser que o servidor acompanhe automaticamente o que estiver atribuído na VNIC. As duas convivem com o mesmo nome de arquivo — escolha uma.

---

## 4. Permissões e SELinux

O NetworkManager só executa scripts do dispatcher que sejam `root:root`, não graváveis por grupo/outros, e executáveis. No Oracle Linux (SELinux em enforcing) o contexto também importa.

```bash
sudo chown root:root /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo chmod 755 /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo restorecon -v /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips 2>/dev/null || true

ls -l /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
```

Esperado: `-rwxr-xr-x. root root` (o `.` indica contexto SELinux aplicado).

---

## 5. Por que estas escolhas (justificativa técnica)

| Decisão | Motivo |
|---|---|
| **Dispatcher** em vez de `rc.local` | `rc.local` roda **uma vez no boot**. O dispatcher dispara em `up` e `dhcp4-change`, cobrindo também reciclagem da interface em runtime (renegociação de DHCP, `nmcli device reapply`). |
| **`ip addr replace`** em vez de `ip addr add` | `replace` é idempotente: não falha nem polui o log se o IP já existir. `add` retorna erro quando o endereço já está presente. (Evita também o falso-positivo de `grep -w`, em que buscar por `192.168.123.6` casaria dentro de `192.168.123.60`.) |
| **Filtro `$IFACE = $DEV`** | O dispatcher dispara para todas as interfaces (inclusive `docker0`); o filtro garante que só agimos na interface alvo. |
| **`restorecon`** | Sem contexto SELinux correto, o NM pode ser impedido de executar o script silenciosamente. |
| **IPs na VNIC primária, não VNICs múltiplas** | Recomendação da Oracle para evitar roteamento assimétrico no Linux. |

---

## 6. Testes

### 6.1. Teste de runtime (reciclagem da interface) — valida o mecanismo

```bash
sudo nmcli device reapply ens3
ip -4 addr show ens3
```

> Rodar o script "na mão" (`sudo .../99-oci-secondary-ips ens3 up`) testa apenas a **lógica** do script, não se o NetworkManager o aciona. Prefira `device reapply` ou reboot.

### 6.2. Conferir presença de todos os IPs

```bash
for ip in 192.168.123.82 192.168.123.141 192.168.123.235 192.168.123.60 192.168.123.206; do
  ip -4 addr show dev ens3 | grep -qw "$ip" && echo "$ip OK na ens3" || echo "$ip AUSENTE"
done
```

### 6.3. Conferir acessibilidade pela rede

> **Importante:** pingar os próprios IPs **a partir da mesma máquina** sempre responde, porque o tráfego não sai da interface — isso confirma apenas que o IP está *atribuído*, não que está *acessível*. Para validar acessibilidade real, faça o ping **de outra máquina da subnet**:
>
> ```bash
> # rodar a partir de OUTRO host na mesma subnet
> for ip in 192.168.123.82 192.168.123.141 192.168.123.235 192.168.123.60 192.168.123.206; do
>   ping -c2 -W1 "$ip" >/dev/null && echo "$ip OK" || echo "$ip FALHA"
> done
> ```

### 6.4. Teste definitivo — reboot

```bash
sudo reboot
```

Ao reconectar:

```bash
ip -4 addr show ens3
nmcli connection show --active
```

O `nmcli` provavelmente mostrará um `Wired Connection` recriado pelo OCI ativo na interface — **isso é esperado**. O que importa é que os IPs secundários apareçam no `ip addr` sem intervenção.

> **Cuidado com o acesso:** se o seu SSH chega por esta interface, tenha a console serial/VNC do OCI à mão antes do reboot. O IP primário volta em segundos, mas é a rede de segurança.

---

## 7. Manutenção (adicionar/remover IPs)

Editar a lista (apenas na versão 3.1; a 3.2 se atualiza sozinha):

```bash
sudo vi /etc/NetworkManager/dispatcher.d/99-oci-secondary-ips
sudo nmcli device reapply ens3
ip -4 addr show ens3
```

Remover um IP que ainda está ativo na interface (o dispatcher não remove sozinho ao tirar da lista):

```bash
sudo ip addr del 192.168.123.82/24 dev ens3
```

---

## 8. Caso B — VNIC secundária (não coberto pelo fluxo acima)

Se os IPs estiverem em uma **VNIC secundária** (placa adicional anexada à instância), o tratamento muda:

- **Caminho oficial Oracle:** usar o utilitário `oci-network-config` (parte do pacote `oci-utils`), recomendado pela Oracle para configurar VNICs secundárias em Oracle Linux. Em RHEL 9.6+/10.0+ há também o `nm-cloud-setup` com `NM_CLOUD_SETUP_OCI=yes`.
- **Policy-based routing:** VNIC secundária exige tabela de rota dedicada e regras de origem, senão o tráfego de retorno sai pela interface errada (roteamento assimétrico). Padrão: `ip rule add from <ip-origem> lookup <tabela>` + rota default na tabela apontando para o `virtualRouterIp` da VNIC.
- **Atenção a iSCSI:** adicionar VNICs pode desviar o tráfego iSCSI da VNIC primária e quebrar volumes anexados. Boot volumes iSCSI usam `169.254.0.2/32` e block volumes usam `169.254.2.0/24`; se necessário, adicione rotas específicas para esses destinos via o roteador da VNIC primária.

Para esse cenário, vale seguir a documentação oficial (`oci-network-config`) em vez do dispatcher manual.

---

## 9. Referências

- **Virtual Network Interface Cards (VNICs)** — visão geral de VNICs, IPs secundários, metadata da instância (IMDS) e configuração do SO para VNICs secundárias:
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm
- **Configuring Linux to Use a Secondary Private IP Address** — orientação oficial sobre persistência de IP secundário no Linux (incluindo o caso de o NetworkManager sobrescrever a configuração no boot):
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIPaddresses_topic-Linux_Details_about_Secondary_IP_Addresses.htm
- **IP Addresses (Private IP Objects)** — comportamento de atribuição de IPs privados e objetos secundários na VNIC:
  https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIPaddresses.htm
- **Instance Metadata Service (IMDS)** — endpoints `/opc/v2/vnics/` usados na versão auto-configurável do script:
  https://docs.oracle.com/iaas/Content/Compute/Tasks/gettingmetadata.htm
- **oci-network-config (oci-utils)** — utilitário oficial Oracle para configuração de VNICs secundárias em Oracle Linux (cenário B):
  https://docs.oracle.com/iaas/oracle-linux/oci-utils/index.htm
