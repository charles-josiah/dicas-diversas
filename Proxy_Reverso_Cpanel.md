# Configuração de Proxy Reverso no cPanel/WHM (Apache)

## Objetivo
Configurar um **Proxy Reverso** para um domínio específico em ambiente **cPanel/WHM**, sem editar diretamente o `httpd.conf`, seguindo o padrão de includes suportado pelo cPanel.

## Ambiente
- **Sistema:** CentOS / CloudLinux / AlmaLinux (cPanel)
- **Servidor Web:** Apache 2.4
- **Módulos Ativos:** `mod_proxy`, `mod_proxy_http`, `mod_proxy_fcgi`, `mod_proxy_wstunnel`
- **Local dos arquivos:** `/etc/apache2/conf.d/userdata/`

## Procedimento

### 1. Confirmar módulos ativos no Apache

```bash
httpd -M | grep proxy
```
Deve listar:
- proxy_module
- proxy_http_module
- proxy_fcgi_module
- proxy_wstunnel_module

---

### 2. Identificar o domínio e o VirtualHost existente

```bash
httpd -S | grep <domínio>
```
Exemplo para `apirev.faznada.com`:

```
port 80 namevhost apirev.faznada.com
port 443 namevhost apirev.faznada.com
```

---

### 3. Criar diretório de include customizado

**Para HTTP (porta 80):**

```bash
mkdir -p /etc/apache2/conf.d/userdata/std/2_4/apirev/apirev.faznada.com/
```

**Para HTTPS (porta 443):**

```bash
mkdir -p /etc/apache2/conf.d/userdata/ssl/2_4/apirev/apirev.faznada.com/
```

---

### 4. Criar arquivo de configuração de proxy reverso

**Exemplo:**
`/etc/apache2/conf.d/userdata/std/2_4/apirev/apirev.faznada.com/reverso.conf`

Conteúdo:

```apache
<IfModule mod_proxy.c>
    ProxyPreserveHost On
    ProxyPass "/" "http://127.0.0.1:8088/"
    ProxyPassReverse "/" "http://127.0.0.1:8088/"
</IfModule>
```

**Para HTTPS** (opcionalmente repetir no caminho `ssl/`).

---

### 5. Aplicar as configurações

Recriar os includes do Apache:

```bash
/scripts/rebuildhttpdconf
```

E depois **reiniciar** o Apache:

```bash
/scripts/restartsrv_httpd
```

ou

```bash
systemctl restart httpd
```

---

### 6. Validar

- Acessar o domínio `http://apirev.faznada.com/` e verificar o comportamento.
**Exemplo de testes usando `curl`:**
```bash
$ curl  https://apirev.faznada.com  
{"data":"25/04/2025 - RELEASE/teste-api-v1.0.4.38","mensagem":"OK.","status":200,"dataHoraServidor":"2025-04-25 08:53:02"}%
$ curl  https://apirev.faznada.com  
{"data":"25/04/2025 - RELEASE/teste-api-v1.0.4.38","mensagem":"OK.","status":200,"dataHoraServidor":"2025-04-25 09:03:12"}%
$ curl  https://apirev.faznada.com  
{"data":"25/04/2025 - RELEASE/teste-api-v1.0.4.38","mensagem":"OK.","status":200,"dataHoraServidor":"2025-04-25 09:24:42"}%
```
- Confirmar funcionamento do HTTPS, se configurado.

---
## Possíveis Problemas

| Problema | Solução |
|:---|:---|
| Proxy não funcionando | Verificar se o Apache foi reiniciado e se o arquivo está no local correto. |
| Proxy quebrando conexão | Validar se o destino interno (127.0.0.1:8088) está UP. |
| Erros 502/504 | Checar firewall e saúde do backend. |

---

## Observações

- Mesmo que o `# Include` esteja comentado nos VirtualHosts, o Apache do cPanel processa **automaticamente** o que estiver em `/etc/apache2/conf.d/userdata/`.
- Sempre use `/scripts/rebuildhttpdconf` antes de restartar o Apache.
- **Não editar** manualmente o `httpd.conf` em servidores cPanel!
