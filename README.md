# Dicas diversas 
Dicas diversas sobre comandos Linux, MACOS, Fortinet, Zimbra, VMWARE e outros.... 

<hr> 
  
**TOPICOS**

[Zimbra](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Zimbra) <br>
[Linux](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Linux)  <br>
[VMWARE](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#VMWARE) <br> 
[MACOS](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#MACOS) <br> 
[Fortinet](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Fortinet) <br> 

<hr>

## Zimbra
* ZIMBRA - Gerar lista com as "Listas de Distribuição" e seus integrantes:
  ````
  for a in `zmprov gadl `; do echo "lista: $a" ; zmprov  gdl $a  | grep zimbraMailForwardingAddress | cut -f2 -d " "; echo "-----";  done 
  ````
  
<hr>

## Linux

* LINUX - systemctl - mostrar serviços ativos
  ````
  systemctl list-unit-files | grep enabled
  ````
* LINUX - RSYNC - Verifica os diretorios no arquivo smb.conf e realiza o sincronismo para outro diretorio.
  ````
  for a in `cat /etc/samba/smb.conf | grep path | grep -v logon | awk '{ print $3} '`; do rsync -Cravzp --progress $a /mnt/backup/ ; done
  ````
* LINUX - RSYNC - Exemplo basico :D  
  ````
  rsync -a -e "ssh -i ~/Acessos/Chave/id_chave_MT" --rsync-path="sudo rsync"  charles.a@189.8.202.54:/vagrant ./
  ````
* LINUX - Ping Multicast em IPv6 - envia uma solicitação de ICMPv6  de 'echo request (type 128)' para 'all-nodes' no endereço multicast. Para ver a vizinhança depois, ip neighbor.
  ````
  ping6 -I <interface> ff02::1
  ip neighbor
  ````
* LINUX - Apresenta Comando, PID da aplicação, a porta e para quem esta realizando a conexão. Ou seja, PID x Aplicacao x Porta usada
  ```` 
  lsof -Pnl +M -i4  (ipv4)
  lsof -Pnl +M -i6  (ipv6) 
  ````  
* LINUX - Mostra porta abertas tcp e udp.
  ````
  netstat -tulpn
  ````
* LINUX - A partir do redhat/centos 7 não temos mais netstat  <h6>Referencia: @NixPal Chris</h6>
 
  | Comando | Descrição |
  | :--- | :--- |
  | ss -s    | (Lista conexões estabelecidas, fechadas, orfã, w esperando fechamento. | 
  | ss -l    | (lista as portas abertas) |
  | ss -pl   | lista as postas abertas e usuarios que a estão utilizando) |
  | ss -t -a | lista todas as portas TCP |
  | ss -u -a | lista todas as portas UDP |
  | ss -w -a | lista portas RAW |
  | ss -x -a | lista todas os Sockets |
  | ss -la -4 | lista conexões em ipv4 | 
  | ss -la -5 | lista conexões em ipv6 | 
  | ss -o state established '( dport = :smtp or sport = :smtp )' | lista portas estabalecias com origem/destino porta smtp | 
  | ss -o state established '( dport = :http or sport = :http )' | lista potras estabalecias com origem/destino porta http | 
  | ss dst/src <ip>:<porta> | lista todas as conexão que o destino X, ache a conexão por exemplo em  ss -la | 
  | ss  sport = :http | lista todas as conexão com origem porta 80 | 
  | ss  dport = :http | lista todas as conexão com destino porta 80 |  
  | ss  dport \&gt; :1024 | lista todas as conexão com destino acima da porta 1024 |  
  | ss  sport \&gt; :1024 | lista todas as conexão com origem acima da porta 1024 |  
  | ss sport \&lt; :32000 | lista todas as conexão com origem menor da porta 32000 | 
  | ss  sport eq :22 | lista todas as conexão com origem origem porta 22 | 
  | ss  dport != :22 | lista todas as conexão com diferente que destino porta 22 | 
  | ss  state connected sport = :http | lista todas as conexão com porta origem 80 | 
  | ss \( sport = :http or sport = :https \) | lista todas as conexão com porta origem 80 OU 443 |   
  | s -o state fin-wait-1 \( sport = :http or sport = :https \) dst <IP> | lista todas as conexão com porta origem 80 OU 443, do statdo fin-wait-1 e com destino <IP> |  

* Linux - debian 9 - rc.local
  - https://www.itechlounge.net/2017/10/linux-how-to-add-rc-local-in-debian-9/
  - https://ritsch.io/2017/08/02/execute-script-at-linux-startup.html

<hr>

## VMWARE 

* VMWARE - Troca da senha de root do HOST VMWare ESXi via VCenter
  <h6>Fonte: https://www.linkedin.com/pulse/reset-esxi-root-password-through-vcenter-esxcli-method-buschhaus/</h6>
  
  ````
  # Inicie um powercli, pode ser via docker: 
  docker pull vmware/powerclicore
  docker run --rm -it vmware/powerclicore

  Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
  Connect-VIServer -Server <IP_VCENTER> -User <ADMINISRADOR_TOP>  -Password <SENHA_ADMIN_TOP>
  $vmhosts = Get-VMHost

  $NewCredential = Get-Credential -UserName "root" -Message "NOVA SENHA QUE VC DESEJA" 

  Foreach ($vmhost in $vmhosts) {
     $esxcli = get-esxcli -vmhost $vmhost -v2 #Gain access to ESXCLI on the host.
     $esxcliargs = $esxcli.system.account.set.CreateArgs() #Get Parameter list (Arguments)
     $esxcliargs.id = $NewCredential.UserName #Specify the user to reset
     $esxcliargs.password = $NewCredential.GetNetworkCredential().Password #Specify the new password
     $esxcliargs.passwordconfirmation = $NewCredential.GetNetworkCredential().Password
     Write-Host ("Resetting password for: " + $vmhost) #Debug line so admin can see what's happening.
     $esxcli.system.account.set.Invoke($esxcliargs) #Run command, if returns "true" it was successful.
  } 

  ````
* VMWARE - Boas praticas de configuração de LUN/Controladora ISCSI no VMWare e Compelent SCv2020
  <h6>Fonte: https://downloads.dell.com/manuals/common/sc-series-vmware-vsphere-best-practices_en-us.pdf </h6>

  ````
  esxcli system module parameters set -p issue_scsi_cmd_to_bringup_drive=0 -m lsi_msgpt3

  for a in `esxcli storage nmp device list | grep ^naa `; do esxcli storage nmp psp roundrobin deviceconfig set --device $a --type=iops --iops=3; done

  esxcli storage nmp satp rule add -s VMW_SATP_ALUA -V COMPELNT -P VMW_PSP_RR -o disable_action_OnRetryErrors -e "Dell EMC SC Series Claim Rule"

  esxcli iscsi adapter param set -A=vmhba64 -k=LoginTimeout -v=5
  esxcli system module parameters set -m iscsi_vmk -p iscsivmk_LunQDepth=255
  ```` 
* VMWARE - LINUX - RedHAT / Centos 6 - Disk Reclain
  ```` 
  reclain area disk  fstrim -v /test
  ```` 
* VMWARE - Ignorar o aviso que o SSH esta ativo no host. 
   <h6>Fonte: https://kb.vmware.com/s/article/2003637 </h6>
   
  ```` 
  vim-cmd hostsvc/advopt/update UserVars.SuppressShellWarning long 1
  ```` 
* VMWARE - Adicionar  Vmware no AD via cli 
  ```` 
  root@srvvcenter01 [ ~ ]# /opt/likewise/bin/domainjoin-cli join  dominio.local usuario
  Joining to AD Domain:   dominio.local
  With Computer DNS Name: srvvcenter01.dominio.local
  ```` 
* VMWARE -  Ligar maquinas via CLI
  <h6> Fonte: http://buildvirtual.net/troubleshooting-esxi-vlan-configurations-using-command-line-tools/ </h6>
  
  ```` 
  # retonar a lista de maquinas com seu ID 
  vim-cmd vmsvc/getallvms
  # podemos usar um grep para mostrar somente a maquina desejada 
  vim-cmd vmsvc/getallvms |grep <vm name>
  # Exibir o status da maquina pelo vmid.
  vim-cmd vmsvc/power.getstate <vmid>
  # Ligar a maquina
  vim-cmd vmsvc/power.on <vmid>
  ```` 

<hr>

## MACOS 

* MACOS - Limpar backups antigos TimeMachine
  ```` 
  for i in /Volumes/BKP/Backups.backupdb/<USER>/2017-01*; do tmutil delete "$i" ; done  
  ```` 
* MACOS - Mostar Arquivos Ocultos
  ```` 
  defaults write com.apple.finder AppleShowAllFiles YES
  killall Finder /System/Library/CoreServices/Finder.app
  defaults write com.apple.finder AppleShowAllFiles NO
  killall Finder /System/Library/CoreServices/Finder.app
  ```` 
* MACOS - Limpar cache USB
  ```` 
  sudo kextunload IOUSBMassStorageClass.kext
  sudo kextload /System/Library/Extensions/IOUSBMassStorageClass.kext
  ```` 
* MACOS - Limpar DNS cache 
  ```` 
  sudo killall -HUP mDNSResponder;sudo killall mDNSResponderHelper;sudo dscacheutil -flushcache
  
  # Exibir a configuração DNS
  scutil --dns
  
  # Scripts Legal:
  https://rakhesh.com/powershell/vpn-client-over-riding-dns-on-macos/

  ```` 
* MACOS - Trocar Hostname
  ````
  sudo scutil --set ComputerName "charles.a"
  sudo scutil --set LocalHostName "charles.a"
  sudo scutil --set HostName "charles.a"
  ````
* MACOS - Brew Link
  ````
  sudo chown -R $(whoami):admin /usr/local  
  brew doctor 
  brew update
  ````
* MACOS - Screen - Utilizando Cabo Console 
  ````
  screen /dev/tty.usbserial  9600
  
  # control-A H - loga toda a saida de comando
  # control-A h - tira print tela texto
  # alias screen='screen /dev/tty.usbserial 9600' - alias para economizar digitação :D 
  ````

<hr>

## Fortinet 
  
* FGT - Reinicia a tabela de roteamento
  ````
  diagnose firewall iprope flush
  ````
* FGT - Reinicia o status do HA
  ````
  diag sys ha reset-uptime
  ````
* FGT - Limpar tabela ARP
  ````
  execute clear system arp table 
  ````
* FGT - Desabilitar SumerTime / Horario de Verão
  ````
  config system global
    set dst disable
  end
  ````
* FGT - DEBUG - VPN Autenticação com AD
  <h6>Fonte: https://forum.fortinet.com/tm.aspx?m=156688 </h6>
  ````
  diagnose debug reset
  diagnose debug app ike -1
  diagnose debug app fnb -1
  diagnose debug enable
  ````
* FGT - Lista de IPs/Usuarios Bloqueados 
  ````
  # usuarios
  get user ban list
  # por hosts
  diagnose firewall ip_host list
  # para remover da lista
  diagnose firewall ip_host rem  src|dst  <ipv4 addr>
  ````
* FGT - Status do HA e reiniciar 1 menbro do cluser
  ````
  get system ha status
  Master:255 FGT01 FGT80Cxxxxxxxxxx 0
  Slave :128 FGT02 FGT80Cxxxxxxxxxx 1
  execute ha manage x
  execute reboot
  ````
* FGT - Forçar,  verificar e "debbugar" atualização na Fortiguard
  ````
  diag deb reset
  diag deb app update -1
  diag deb enable
  execute update-now
  # Desligar debug 
  diag deb dis
  diag deb reset
  ````
* FGT - Status da "caixa" e do Cluster HA
  ````
  get system status
  diagnose sys ha status
  di sys ha cluster-csum
  ````
* FGT - CPU e Memoria monitoramento
  ````
  diagnose sys top
  # diag sys kill 11 - matar o vilão 
  # ficar de olho: guacd e wad sao vilões de memoria

  diagnose sys top-summary
  # Press "m" to sort by memory
  # Press "c" to sort by cpu
* FGT - Ler log do ultimo crash no equipamento
  ````
  diagnose debug crashlog read
  ````
* FGT - Fortinet Criptografia Alta  
  ````
  config system global
     set strong-crypto enable
     set admin-https-ssl-versions tlsv1-2
  end

  config firewall ssl setting
      set ssl-dh-bits 2048
  end

  config vpn ssl settings
  set sslv3 disable
      set tlsv1-0 disable
      set tlsv1-1 disable
      set algorithm high
      edit 1
           set cipher high
      end
  end
  ````
* FGT - FSSO - debugs
  ````
  diag deb authd fsso <comands>
  diagnose firewall auth  list

  # para regras já existentes, bug que precisa ativar o fsso na mão, set fsso enable 
  ````
* FGT - FSSO - Limpar Cache e aplicação DNS
  <h6> http://kb.fortinet.com/kb/viewContent.do?externalId=FD30618 </h6> 
  
  ````
  diag test application dnsproxy 1
  diag test application dnsproxy ? # ver outros niveis de teste
  ````

 
 

<h6>
Obs.: Maioria destes comandos foram utilizados para resolver problemas pontuais, a alguns são de muito, muito tempo atrás, não possuem nenhuma "boniteza" e organização nos mesmos. São mais como notas para não esquecimento :D 
</h6>
