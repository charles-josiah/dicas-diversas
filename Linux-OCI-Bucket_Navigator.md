
# ☁️ OCI Bucket Navigator — Guia & Função `oci_download`

> 🇬🇧 **English version:** [Linux-OCI-Bucket_Navigator.en.md](Linux-OCI-Bucket_Navigator.en.md)

Este guia documenta uma função de *shell* para **listar, indexar e baixar** objetos de um bucket da **Oracle Cloud Infrastructure (OCI)** diretamente do terminal, escolhendo o arquivo pelo **índice** exibido na listagem.

> **Por que colocar no `.bashrc`?**  
> Ao adicionar as `exports` e a função `oci_download` no seu `~/.bashrc` (ou `~/.bash_profile`), elas ficam **disponíveis em todas as sessões** do seu usuário sem precisar copiar/colar a cada vez. Basta abrir um novo terminal (ou `source ~/.bashrc`) e usar o comando.

---

## ✅ Pré‑requisitos

- **OCI CLI** instalado e autenticado (`oci setup config`)  
- **jq** para tratar JSON  
- Permissões no bucket (policy adequada na tenancy/compartment)  
- Acesso de rede ao endpoint do Object Storage

---

## 🔧 Variáveis de ambiente

Adicione no seu `~/.bashrc` (ajustando os valores):

```bash
# Silencia warnings de libs antigas do Python/OCI CLI
export PYTHONWARNINGS="ignore"

# Identificação do bucket na OCI
export BUCKET_NAME="SEU_BUCKET"
export NAMESPACE="SEU_NAMESPACE"
```

> Dica: se você usa múltiplos ambientes, considere exportar também `OCI_CLI_PROFILE` (ex.: `dev`, `prod`) para alternar perfis do `~/.oci/config`.

Após editar o `.bashrc`, recarregue:
```bash
source ~/.bashrc
```

---

## 🧩 Função `oci_download` (com índices)

Cole a função abaixo no **mesmo `.bashrc`** (ou em um arquivo de funções e `source` dele):

```bash
oci_download() {

  local idx="${1:-}"
  local dest="${2:-/tmp}"
  local obj_name out

  echo "=== Lista de objetos no bucket $BUCKET_NAME ==="
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
  read -rp "Digite o índice do objeto a baixar ou q|Q para sair: " idx
  if [[ "$idx" == "q" || "$idx" == "Q" ]]; then
    echo "Saindo sem baixar nada..."
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
    echo "Índice inválido!"
    return 1
  fi

  mkdir -p "$dest"
  out="$dest/$(basename -- "$obj_name")"
  echo "Baixando: $obj_name -> $out"

  oci os object get \
    --namespace-name "$NAMESPACE" \
    --bucket-name "$BUCKET_NAME" \
    --name "$obj_name" \
    --file "$out"
}
```

**O que ela faz?**
1. Chama `oci os object list` e transforma o JSON em uma **tabela numerada** (via `jq` + `awk`), onde a primeira coluna é o **índice** (`.key` da entrada).  
2. Pede ao usuário um **índice**; `q`/`Q` sai sem baixar.  
3. Resolve o **nome do objeto** pela posição `.data[$idx].name`.  
4. Faz o download com `oci os object get` para `dest` (padrão: `/tmp`).

---

## ▶️ Exemplos de uso

Listar e baixar (modo interativo):
```bash
oci_download
# ... a função lista objetos e pergunta: "Digite o índice..."
```

Passando pasta de destino (mantém o prompt de índice):
```bash
oci_download "" ~/Downloads
```

Baixar e salvar em `/var/tmp` (com prompt):
```bash
oci_download "" /var/tmp
```

> A função é **interativa** por design: mesmo com parâmetros, ela sempre pergunta o índice para evitar enganos ao baixar vários objetos.

---

## 🧪 Exemplo de saída (listagem)

```
    0 |  12.54 MB | 2025-09-15T10:12:34.000Z | backups/db-snapshot-2025-09-15.sql.gz
    1 | 803.00 KB | 2025-09-14T22:01:10.000Z | logs/app-2025-09-14.log.gz
    2 |   2.10 GB | 2025-09-14T01:22:00.000Z | images/disk.qcow2
```

---

## 🧠 Por que no `.bashrc`?

- **Disponível sempre**: você ganha um “comando” novo no shell para navegar/baixar do bucket.  
- **Padroniza ambiente**: as variáveis `BUCKET_NAME`/`NAMESPACE` ficam definidas para todos os terminais.  
- **Menos atrito**: evita repetir export/definições e reduz erros operacionais.  

> Para instalar para **todos os usuários**, coloque num arquivo em `/etc/profile.d/oci_download.sh` com as `exports` e a função, e defina permissões apropriadas.

---

## 🛡️ Boas práticas & observações

- **Perfis**: use `export OCI_CLI_PROFILE=prod` para travar o perfil corrente.  
- **Cotas/latência**: `--all` pagina tudo; em buckets enormes pode demorar. Se necessário, filtre com `--prefix` e/ou troque para paginação manual.  
- **Segurança**: não versionar `~/.oci/config` com chaves; use **Vault**/KMS ou variáveis sob controle.  
- **Erros comuns**:  
  - `NotAuthorizedOrNotFound`: checar policy do user/dynamic group no compartment.  
  - `jq: command not found`: instalar `jq`.  
  - `ObjectNotFound`: o objeto pode ter sido removido entre a listagem e o *get* (concorrência).

---

## 🔁 Variações úteis (opcional)

- **Filtro por prefixo** (ex.: apenas `logs/`):
  ```bash
  oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET_NAME" --prefix "logs/"
  ```
- **Baixar sem prompt** (quando já sabe o nome exato):
  ```bash
  oci os object get --namespace-name "$NAMESPACE" --bucket-name "$BUCKET_NAME" --name "caminho/arquivo.ext" --file "/tmp/arquivo.ext"
  ```

---

## 🧷 Checklist rápido de instalação

1. Instalar `oci` e `jq`.  
2. Configurar `oci setup config`.  
3. Editar `~/.bashrc`: adicionar `exports` e a função `oci_download`.  
4. `source ~/.bashrc`.  
5. Rodar `oci_download` e escolher o índice.


:wq!
