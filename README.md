# Dicas diversas 
Dicas diversas sobre comandos linux, MACOS, Fortinet, Zimbra, e outros.... 

**TOPICOS**

[Zimbra](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Zimbra) <br>
[Linux](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Linux)  <br>
[VMWARE](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#VMWARE) <br> 
[MACOS](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#MACOS) <br> 

<hr>

## Zimbra
* ZIMBRA - Gerar lista com as "Listas de Distribuição" e seus integrantes:
  ````
  for a in `zmprov gadl `; do echo "lista: $a" ; zmprov  gdl $a  | grep zimbraMailForwardingAddress | cut -f2 -d " "; echo "-----";  done 
  ````

## Linux

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







<h6>
Obs.: Maioria destes comandos foram utilizados para resolver problemas pontuais, a alguns são de muito, muito tempo atrás, não possuem nenhuma "boniteza" e organização nos mesmos. São mais como notas para não esquecimento :D 
</h6>
