# 📚 Linux - Monitor Memoria e Swap


## 💾 Monitoramento de Memória e Swap

### 🔹 Consumo geral
- **Por usuário (smem)**
  ```bash
  sudo smem -rs swap -tk -u
  ```
  > Mostra estatísticas de swap, USS, PSS e RSS agrupadas por usuário.  
  Útil para identificar quem está consumindo mais memória/swap.  

- **Visão geral (GB)**
  ```bash
  free -g
  ```
  > Resumo do uso de memória RAM e swap no sistema, em gigabytes.  

---

### 🔹 Script: `mem.sh`  
**Descrição:**  
Lista o uso de **swap por usuário**, em MB e percentual do total de swap do sistema.  
Ajuda a identificar usuários que mais consomem swap em servidores compartilhados.  

**Código comentado:**  
```bash
#!/bin/bash
# Obtém o total de swap disponível no sistema em KB
TOTAL_SWAP_KB=$(free -k | awk '/^Swap:/{print $2}')
[ -z "$TOTAL_SWAP_KB" ] && TOTAL_SWAP_KB=1

# Cabeçalho da saída
echo -e "User	Swap(MB)	%Swap"

# Itera por cada usuário que possui processos rodando
for U in $(ps -eo user= | awk '{print $1}' | sort -u); do
  SUM_KB=0
  # Itera por cada processo do usuário
  for PID in $(pgrep -u "$U"); do
    if [ -r "/proc/$PID/smaps" ]; then
      # Tenta somar SwapPss (mais preciso)
      KB=$(awk '/^SwapPss:/{s+=$2} END{print s+0}' /proc/$PID/smaps)
      # Se SwapPss não existir, usa VmSwap do /proc/[pid]/status
      if [ "$KB" -eq 0 ]; then
        KB=$(awk '/^VmSwap:/{s+=$2} END{print s+0}' /proc/$PID/status 2>/dev/null)
      fi
      SUM_KB=$((SUM_KB + KB))
    fi
  done
  # Converte KB → MB
  MB=$(( (SUM_KB+1023) / 1024 ))
  # Calcula percentual de swap usado pelo usuário
  PCT=$(awk -v s=$SUM_KB -v t=$TOTAL_SWAP_KB 'BEGIN{printf "%.2f", (t>0 ? (s/t*100) : 0)}')
  echo -e "$U	$MB		$PCT%"
done
```

**Uso:**  
```bash
chmod +x mem.sh
./mem.sh
```

**Exemplo de saída:**  
```
User    Swap(MB)    %Swap
oracle  70          5.3%
mysql   59          4.4%
root    746         10.2%
```

---

### 🔹 Script: `mem_user.sh`  
**Descrição:**  
Mostra os **processos de um usuário específico que estão consumindo swap**.  
Pode exibir apenas o comando (`comm`) ou o comando completo (`args`).  

**Código comentado:**  
```bash
#!/bin/bash
# Uso: ./mem_user.sh usuario [-c]

USER_TARGET=$1     # Nome do usuário
SHOW_CMD=0         # Flag para exibir comando completo

# Se usuário não informado, mostra ajuda
if [ -z "$USER_TARGET" ]; then
  echo "Uso: $0 usuario [-c]"
  exit 1
fi

# Se passada a flag "-c" ou "--cmd", ativa saída detalhada
if [ "$2" == "-c" ] || [ "$2" == "--cmd" ]; then
  SHOW_CMD=1
fi

# Cabeçalho da saída
if [ $SHOW_CMD -eq 1 ]; then
  echo -e "PID	Swap(MB)	Comando Completo"
else
  echo -e "PID	Swap(MB)	Comando"
fi

# Itera pelos processos do usuário
for pid in $(pgrep -u "$USER_TARGET"); do
    if [ -r "/proc/$pid/status" ]; then
        # Captura o consumo de swap do processo
        swap=$(grep VmSwap /proc/$pid/status 2>/dev/null | awk '{print $2}')
        if [ -n "$swap" ] && [ "$swap" -gt 0 ]; then
            # Decide se mostra apenas nome do comando ou linha completa
            if [ $SHOW_CMD -eq 1 ]; then
                cmd=$(ps -p $pid -o args= 2>/dev/null)
            else
                cmd=$(ps -p $pid -o comm= 2>/dev/null)
            fi
            mb=$((swap/1024))
            echo -e "$pid	$mb		$cmd"
        fi
    fi
done | sort -k2 -nr   # Ordena por consumo de swap (maior primeiro)
```

**Uso:**  
```bash
chmod +x mem_user.sh
./mem_user.sh oracle
./mem_user.sh oracle -c   # mostra comandos completos
```

**Exemplo de saída:**  
```
PID     Swap(MB)   Comando
12345   512        java
23456   128        mysqld
```

---

