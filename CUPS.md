# 🖨️ Impressoras CUPS - Guia de Monitoramento e Diagnóstico

Este guia traz **dicas práticas** e um **script em Bash** para auxiliar na administração de impressoras gerenciadas pelo **CUPS (Common UNIX Printing System)**.  

O material cobre desde a **checagem de conectividade das impressoras** até o **diagnóstico de memória, spool, logs e conexões** do serviço `cupsd`.  

---

## 📌 Objetivos

- Identificar impressoras cadastradas no servidor CUPS.  
- Extrair o endereço IP associado a cada impressora.  
- Testar conectividade e exibir status **ONLINE** ou **OFFLINE**.  
- Monitorar uso de memória, spool e conexões do `cupsd`.  
- Fornecer comandos adicionais de troubleshooting.  

---

## 🖥️ Monitoramento do serviço `cupsd`

Ferramentas para diagnóstico rápido do processo principal e filas de impressão:  

### 🔹 Uso de memória e CPU
```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | grep cups
```

### 🔹 Diagnóstico detalhado do processo
```bash
PID=$(pidof cupsd)
echo "PID=$PID"

echo "== STATUS =="; egrep 'VmRSS|VmSize' /proc/$PID/status
echo "== PMAP =="; pmap -x $PID | tail -n 5
echo "== DELETED =="; lsof -p $PID | grep deleted || echo "sem deleted"

echo "== SPOOL =="; du -sh /var/spool/cups; find /var/spool/cups -type f | wc -l
echo "== JOBS =="; lpstat -W not-completed 2>/dev/null | head -n 5
echo "== FILTERS =="; ps aux | egrep 'foomatic|ghostscript|gs|pdftopdf|rasterto|pstops' | egrep -v grep || echo "sem filtros ativos"
echo "== CONNS =="; ss -tp state established '( sport = :ipp or sport = :http )' || true
echo "== LOGLEVEL =="; grep -i '^LogLevel' /etc/cups/cupsd.conf || echo "padrão"
```

---

## 📂 Script de Teste de Impressoras

```bash
#!/bin/bash

# Lista todas as impressoras e extrai IPs
lpstat -v | while read -r line; do
    nome=$(echo "$line" | awk '{print $3}' | sed 's/:$//')
    url=$(echo "$line" | awk '{print $4}')
    ip=$(echo "$url" | grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}')

    if [ -n "$ip" ]; then
        echo -n "Impressora: $nome | IP: $ip -> "

        if ping -c 2 -W 2 "$ip" > /dev/null 2>&1; then
            echo "ONLINE"
        else
            echo "OFFLINE"
        fi
    fi
done
```

---

## 🔎 Explicação passo a passo

1. **Listagem das impressoras no CUPS**  
   ```bash
   lpstat -v
   ```
   Exibe todas as impressoras e seus URIs.  

   Exemplo:  
   ```
   device for HP_LaserJet: socket://192.168.0.50
   device for Epson_Wifi: ipp://192.168.0.60/ipp/print
   ```

2. **Captura do nome da impressora**  
   ```bash
   awk '{print $3}' | sed 's/:$//'
   ```
   Extrai o nome (`HP_LaserJet`) e remove os `:` finais.  

3. **Captura da URL de conexão**  
   ```bash
   awk '{print $4}'
   ```
   Retorna a URL da impressora (`socket://192.168.0.50`).  

4. **Extração do IP**  
   ```bash
   grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}'
   ```
   Identifica o endereço IPv4 presente na URL.  

5. **Teste de conectividade**  
   ```bash
   ping -c 2 -W 2 $ip
   ```
   - `-c 2`: envia 2 pacotes ICMP.  
   - `-W 2`: timeout de 2 segundos.  

   Impressoras que respondem = **ONLINE**, caso contrário = **OFFLINE**.  

---

## 🛠️ Comandos CUPS relevantes

- **`lpstat -v`** → Lista impressoras e URIs de dispositivos.  
- **`lpstat -p`** → Exibe impressoras e status (ativa/parada).  
- **`lpstat -d`** → Mostra impressora padrão.  
- **`lpstat -o`** → Lista jobs na fila de impressão.  
- **`lpstat -t`** → Exibe resumo completo (status, jobs, impressora padrão).  
- **`lpq -a`** → Mostra todas as filas e jobs ativos.  

---

## 🔧 Troubleshooting Avançado

### 🔹 Logs detalhados do CUPS
```bash
# Editar /etc/cups/cupsd.conf
LogLevel debug2

# Reiniciar serviço
systemctl restart cups

# Acompanhar logs
tail -f /var/log/cups/error_log
```

### 🔹 Testar impressora diretamente via URI
```bash
ipptool -tv ipp://192.168.0.60/ipp/print get-printer-attributes.test
```

### 🔹 Enviar job de teste
```bash
echo "Página de Teste via CUPS" | lp -d HP_LaserJet
```

### 🔹 Listar drivers e filtros disponíveis
```bash
lpinfo -m
```

### 🔹 Verificar portas e firewall
```bash
ss -lntp | grep -E '631|9100|515'
```
- IPP → `631/tcp`  
- RAW/Socket → `9100/tcp`  
- LPD → `515/tcp`  

### 🔹 Monitoramento SNMP
```bash
snmpwalk -v1 -c public 192.168.0.50 1.3.6.1.2.1.43.10.2.1.4
```
👉 Exibe contadores de páginas impressas.  

### 🔹 Teste SSL/TLS
```bash
openssl s_client -connect 192.168.0.60:443
```

### 🔹 Dump de requisições IPP
```bash
tcpdump -i any port 631 -A
```

---

## 📌 Exemplo de saída

```bash
Impressora: HP_LaserJet | IP: 192.168.0.50 -> ONLINE
Impressora: Epson_Wifi | IP: 192.168.0.60 -> OFFLINE
```

---

## ✅ Benefícios

- Automatiza a checagem de impressoras em rede.  
- Mostra **nome, IP e status de conectividade** em uma linha.  
- Fornece ferramentas extras para troubleshooting completo.  
- Permite diagnosticar desde problemas de rede até filtros e jobs travados.  

---

## 🚀 Como usar

1. Salve o script como `test_impressora.sh`.  
2. Torne-o executável:  
   ```bash
   chmod +x test_impressora.sh
   ```  
3. Execute:  
   ```bash
   ./test_impressora.sh
   ```

---
