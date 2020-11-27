# Dicas diversas 
Dicas diversas sobre comandos Linux, MACOS, Fortinet, Zimbra, VMWARE, Zabbix e outros.... 

<hr> 
  
**TOPICOS**

[Zimbra](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Zimbra) <br>
[Linux](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Linux)  <br>
[VMWARE](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#VMWARE) <br> 
[MACOS](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#MACOS) <br> 
[Fortinet](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Fortinet) <br> 
[Zabbix](https://github.com/charles-josiah/dicas-diversas/blob/master/README.md#Zabbix) <br> 


<hr>

## Zimbra
* ZIMBRA - Gerar lista com as "Listas de Distribuição" e seus integrantes:
  ````
  for a in `zmprov gadl `; do echo "lista: $a" ; zmprov  gdl $a  | grep zimbraMailForwardingAddress | cut -f2 -d " "; echo "-----";  done 
  ````
* ZIMBRA - Mostrar as contador de mensagens e pastas de um usuario via CLI.
  ````
  zmmailbox -z -m <usuario@domino> gaf | more
  ````
* ZIMBRA - Mostrar contador <b>aproximado</b> de mensagem total da caixa de um usuario. <b>Aproximado</b> porque, soma shared-folders e outros itens junto, precisa melhorar o filtro para somar somente mensagens.
  ````
  total=0; for a in `zmmailbox -z -m  gaf <usuario@domino> | egrep "mess|unkn" | grep -vi contac  |  awk '{ print $4 }' | egrep -o "[0-9]+" ;`; do total=$(( total + a )); done; echo "$total"
  ````
* ZIMBRA - Mostrar contador <b>aproximado</b> de mensagem total da caixa de um usuario. <b>Aproximado</b> porque, soma shared-folders e outros itens junto, precisa melhorar o filtro para somar somente mensagens. Porem adicionado para um dominio todo.
  ````
  for b in `zmprov -l gaa | grep -v spam | grep -v ham | grep -v galsync | grep -v virus | grep <DOMINIO> | cut -d "@" -f1`; do total=0; for a in `zmmailbox -z -m  $b@b<DOMINO> gaf  |  egrep "mess|unkn"  | awk '{ print $4 }' | egrep -o "[0-9]+";`; do total=$(( total + a )); done; echo "$b;$total"; done 
  ````
* ZIMBRA - Erro de TLS quando inicia o servidor zimbra -  Unable to start TLS: .
  ````
  log: Unable to start TLS: hostname verification failed when connecting to ldap master.
  [zimbra@zimbra ssl]$ zmlocalconfig -e ldap_starttls_required=false
  [zimbra@zimbra ssl]$ zmlocalconfig -e ldap_starttls_supported=0

  ````
* ZIMBRA - Recuperar "encaminhador" de mensagens da conta do usuario e preparar a saida o comando, gerando um comando para reinserir em um outro zimbra . 
  ````
  for a in `zmprov -l gaa | grep <DOMINIO>`; do  echo -n "zmprov ma $a zimbraPrefMailForwardingAddress  \" `  zmprov  ga $a  | grep zimbraPrefMailForwardingAddress | sed s/\zimbraPrefMailForwardingAddress:// ` \"  "; echo; done
  ````
* ZIMBRA - Localizar uma mensagem via mysql e cli.
    - Fonte: https://wiki.zimbra.com/wiki/Account_mailbox_database_structure
  
* ZIMBRA - Recuperar mensagem .msg via cli.
    - Fonte: https://www.sv.net.br/zimbra-reimportar-arquivos-msg/
  ````
  zmmailbox -z -m  <email>  addMessage /<IMAP DIR>  1783-1607.msg   
  
  ````
* ZIMBRA - Exibir todas as sessões ativas. E mata-lás !
  ````
  echo "exibir: "
  
  /opt/zimbra/bin/zmsoap -z -v GetSessionsRequest @type=soap  #sessoes web
  /opt/zimbra/bin/zmsoap -z -v GetSessionsRequest @type=imap  #sessoes imap
  /opt/zimbra/bin/zmsoap -z -v GetSessionsRequest @type=admin #sessoes do admin
  
  echo "Vamos invalidar todas:"
  for a in `/opt/zimbra/bin/zmsoap -z -v GetSessionsRequest @type=soap | awk '{ print $5 } ' | sed s/name=// | sed s/\"//g` ; do zmprov ma $a  zimbraAuthTokenValidityValue  1; done
  ````
* ZIMBRA - Ao iniciar o zimbra apresentando erro de conexão TLS ao acessão a arvore LDAP.
  ```` 
  Unable to start TLS: SSL connect attempt failed error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed when connecting to ldap master.
  [zimbra@mail ~]$ zmlocalconfig -e ldap_starttls_required=false
  [zimbra@mail ~]$ zmlocalconfig -e ldap_starttls_supported=0
  ````
 <hr>
 
## Linux
* LINUX - Serviço na porta 5535 - Link-Local Multicast Name Resolution - Centos/Ubuntu
  ````
   cat  /etc/systemd/resolved.conf 
      #  This file is part of systemd.
      #
      #  systemd is free software; you can redistribute it and/or modify it
      #  under the terms of the GNU Lesser General Public License as published by
      #  the Free Software Foundation; either version 2.1 of the License, or
      #  (at your option) any later version.
      #
      # Entries in this file show the compile time defaults.
      # You can change settings by editing this file.
      # Defaults can be restored by simply deleting this file.
      #
      # See resolved.conf(5) for details

      [Resolve]
      #DNS=
      #FallbackDNS=
      #Domains=
      LLMNR=no            #   <--------- ALTERAR de YES para NO, e init 6
      #MulticastDNS=yes
      #DNSSEC=allow-downgrade
      #DNSOverTLS=no
      #Cache=yes
      #DNSStubListener=udp
  ````
* LINUX - Tunel SOCKS via SSH. Tunel estará disponivel no porta 2611 na localhost. So configurar o browser, em proxy para esta porta.
  ````
  ssh -v  -D 2611 -C -q -N  cha.rles.xyz
  ````
* LINUX - Tunel SSH para "exportar" uma porta remota para uma porta na sua estação local
  ````
  Terminal local 1:  ssh -L 2222:172.31.16.2:22 ch@rles.xyz
  Terminal local 2:  ssh charles.a@localhost -p 2222
  ````
* LINUX - Tunel SSH para "exportar" uma porta do seu servidor/estação local para um servidor remoto (muito bom para nao precisar fazer NAT)
  ````
  Terminal local 1:  ssh -R 2222:127.0.0.1:22 ch@rles.xyz
  Terminal remoto 1:  ssh charles.a@localhost -p 2222
  ````
* LINUX - Limpar tabela MAC
  ````
  ip -s -s neigh flush all
  ````
* LINUX - mostar interface e ipv4 das placas de rede
  ````
  #exemplo
  $ echo; for a in `ip addr show | grep mtu | awk '{print substr($2, 1, length($2)-1)}'`; do echo -n "Placa: $a - IP: "; ip -4 -o addr show $a | awk '{print $4}'; done; echo 

  Placa: lo - IP: 127.0.0.1/8
  Placa: eth1 - IP: 10.56.12.14/24
  $
  $
  ````
  
* LINUX - operações com SED inline
  ````
  sed -i.bkp   '1,/\ \ user:..*/s/\ \ user:..*/  user: charles.a/' *.yaml     #encontra o primeiro "match" e realiza a substituição
  sed -i.bkp-hosts   "s/\-\ \hosts:..*/\-\ hosts: all/g" *.yaml               #encontra todos os "hosts" e altera para all
  ````

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

* Linux - Centos 7 - 100% CPU yumBackend.py
  ````
  systemctl stop packagekit-offline-update.service
  systemctl disable packagekit-offline-update.service
  ````
* Linux - Teste de IOPS no linux
  ````
  fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
  ````
<hr>

## VMWARE 

* VMWARE - Disco e arquivos orfaos //  Orphaned files and folders // Zombie Files
  - Fonte: https://www.lucd.info/2016/09/13/orphaned-files-revisited/
  - Fonte: https://virtualfrog.wordpress.com/2017/07/25/powercli-find-zombie-files-on-datastores/
  - Fonte: https://communities.vmware.com/thread/553595
  ````
  # salve o script desejado, no meu caso mosta_odata.ps1, em um diretorio e inicie como volume do docker
  
  Inicie um powercli, pode ser via docker: 
  docker pull vmware/powerclicore
  docker run --rm -v /root/scripts/:/tmp -it vmware/powerclicore 
  
  #carrege o script com a função
  PS /tmp> . ./mosta_odata.ps1
  
  #conectando no vcenter
  PS /tmp> Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
  PS /tmp> Connect-VIServer -Server <IP_VCENTER> -User <ADMINISRADOR_TOP>  -Password <SENHA_ADMIN_TOP>
  PS /tmp> Get-Cluster -Name "NOME_DO_CLUSTER_HA"
  #
  #
  PS /tmp> 
  PS /tmp> Get-Cluster -Name NOME-Cluster | get-datastore | Get-VmwOrphan
  PS /tmp> 
  #
  #
  #para saida no formato xls
  #requer: PS /tmp> Install-Module -Name ImportExcel
           PS /tmp> Update-Module -Name ImportExcel  
           root@debian:~/scripts# apt-get -y update && apt-get install -y --no-install-recommends libgdiplus libc6-dev         
           
  PS /tmp> $reportName = './discos.xls'
  PS /tmp> foreach($ds in (Get-Cluster -Name MyCluster | Get-Datastore | Get-VmwOrphan | Group-Object -Property {$_.Folder.Split(']')[0].TrimStart('[')})){$ds.Group | Export-Excel -Path $reportName -WorkSheetname $ds.Name -AutoSize -AutoFilter -FreezeTopRow }
  PS /tmp> 
  PS /tmp>
  ````
  
* VMWARE - Ver status vmware-tools no ambiente, se instalado/à instalar e e atualizar.
  - Fonte: http://blog.jgriffiths.org/powercli-how-to-get-vmware-tools-version/
  ````
  # Inicie um powercli, pode ser via docker: 
  docker pull vmware/powerclicore
  docker run --rm -it vmware/powerclicore

  Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
  Connect-VIServer -Server <IP_VCENTER> -User <ADMINISRADOR_TOP>  -Password <SENHA_ADMIN_TOP>
  
  #status do vmwware-tools de todas as maquinas no ambiente
  get-vm | % { get-view $_.id } | select Name, @{ Name="ToolsVersion"; Expression={$_.config.tools.toolsVersion}},@{ Name="ToolStatus"; Expression={$_.Guest.ToolsVersionStatus}}
  
  #status do vmware-tools de cada maquina ligada.
  get-vm | where {$_.powerstate -ne "PoweredOff" } | % { get-view $_.id } | select Name, @{ Name="ToolsVersion"; Expression={$_.config.tools.toolsVersion}},@{ Name="ToolStatus"; Expression={$_.Guest.ToolsVersionStatus}}
  
  #status do vmware-tools que não esta atualizado da maquinas ligadas
  get-vm | where {$_.powerstate -ne "PoweredOff" } | where {$_.Guest.ToolsVersionStatus -ne "guestToolsCurrent"} | % { get-view $_.id } | select Name, @{ Name="ToolsVersion"; Expression={$_.config.tools.toolsVersion}}, @{ Name="ToolStatus"; Expression={$_.Guest.ToolsVersionStatus}}

  #status do vmware-tools das maquinas ligadas e não atualizadas com saida para um arquivo csv.
  get-vm | where {$_.powerstate -ne "PoweredOff" } | where {$_.Guest.ToolsVersionStatus -ne "guestToolsCurrent"} | % { get-view $_.id } | select Name, @{ Name="ToolsVersion"; Expression={$_.config.tools.toolsVersion}}, @{ Name="ToolStatus"; Expression={$_.Guest.ToolsVersionStatus}} | Export-Csv -NoTypeInformation -UseCulture -Path /tmp/VMHWandToolsInfo.csv
  ````

* VMWARE - Troca da senha de root do HOST VMWare ESXi via VCenter
  - Fonte: https://www.linkedin.com/pulse/reset-esxi-root-password-through-vcenter-esxcli-method-buschhaus/
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
  
* VMWARE - Mostrar a data de instalação de um HOST Vmware ESXi via VCenter
  - Fonte: https://communities.vmware.com/thread/558662
  ````
  # Inicie um powercli, pode ser via docker: 
  docker pull vmware/powerclicore
  docker run --rm -it vmware/powerclicore
   
  Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
  Connect-VIServer -Server <IP_VCENTER> -User <ADMINISRADOR_TOP>  -Password <SENHA_ADMIN_TOP>
  $vmhosts = Get-VMHost 

  Get-VMHost  | %{
    $thisUUID = (Get-EsxCli -VMHost $_.name).system.uuid.get()
    $decDate = [Convert]::ToInt32($thisUUID.Split("-")[0], 16)
    $installDate = [timezone]::CurrentTimeZone.ToLocalTime(([datetime]'1/1/1970').AddSeconds($decDate))
    [pscustomobject][ordered]@{
    Name="$($_.name)"
    InstallDate=$installDate
   } # end custom object
  } # end host loop
  ````
* VMWARE - Boas praticas de configuração de LUN/Controladora ISCSI no VMWare e Compelent SCv2020
  - Fonte: https://downloads.dell.com/manuals/common/sc-series-vmware-vsphere-best-practices_en-us.pdf
  ````
  esxcli system module parameters set -p issue_scsi_cmd_to_bringup_drive=0 -m lsi_msgpt3

  for a in `esxcli storage nmp device list | grep ^naa `; do esxcli storage nmp psp roundrobin deviceconfig set --device $a --type=iops --iops=3; done

  esxcli storage nmp satp rule add -s VMW_SATP_ALUA -V COMPELNT -P VMW_PSP_RR -o disable_action_OnRetryErrors -e "Dell EMC SC Series Claim Rule"

  esxcli iscsi adapter param set -A=vmhba64 -k=LoginTimeout -v=5
  esxcli system module parameters set -m iscsi_vmk -p iscsivmk_LunQDepth=255
  ```` 
* VMWARE - Troca do driver da placa de rede Broadcom no VMWARE 6.x - Problema de Latencia extrema e perda de pacotes na rede LAN e SAN
  ```` 
   esxcli system maintenanceMode set –enable=true  #procedimento deve ser realizado em modo manutenção.             
   esxcfg-nics -l                                  #exibe os drivers que estão sendo utilizados
   esxcfg-module -d ntg3                           #desabilita o driver ntg3
   esxcfg-module -e tg3                            #habilita o driver tg3
   init 6                                          #reboot
   esxcli system maintenanceMode set --enable=false  #sair do modo manutenção     
   ```` 
* VMWARE - Para melhorar performace rede SAN ISCSI desabilitando "delayed ack"
  - Fonte: https://kb.vmware.com/s/article/1002598
  - script1: http://blog.gptnet.net/?p=421
  - script2: https://tech.zsoldier.com/2011/09/disable-delayed-acknowledgement-setting.html
  `````
  #This section will get host information needed
  $HostView = Get-VMHost NameofESXServer | Get-View
  $HostStorageSystemID = $HostView.configmanager.StorageSystem
  $HostiSCSISoftwareAdapterHBAID = ($HostView.config.storagedevice.HostBusAdapter | where {$_.Model -match "iSCSI Software"}).device

  #This section sets the option you want.
  $options = New-Object VMWare.Vim.HostInternetScsiHbaParamValue[] (1)

  $options[0] = New-Object VMware.Vim.HostInternetScsiHbaParamValue
  $options[0].key = "DelayedAck"
  $options[0].value = $false

  #This section applies the options above to the host you got the information from.
  $HostStorageSystem = Get-View -ID $HostStorageSystemID
  $HostStorageSystem.UpdateInternetScsiAdvancedOptions($HostiSCSISoftwareAdapterHBAID, $null, $options)
  ```` 
* VMWARE - LINUX - RedHAT / Centos 6 - Disk Reclain
  ```` 
  reclain area disk  fstrim -v /test
  ```` 
* VMWARE - Ignorar o aviso que o SSH esta ativo no host. 
  - Fonte: https://kb.vmware.com/s/article/2003637
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
  - Fonte: http://buildvirtual.net/troubleshooting-esxi-vlan-configurations-using-command-line-tools/
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
* VMWARE -  Desbloqueando usuario depois de inumeras vezes a senha errada 
  ```` 
  Procedure to unlock the ESXi host account at the console
  At the console press CTRL+ALT+F2 to get to the ESXi shell. If a login shows up continue with step 3, otherwise continue with step 2.
  Login to the DCUI (to enable the ESXi Shell if not already done)
  Login with root and the correct password.
  Go to Troubleshooting Options
  Select Enable ESXi Shell
  Press CTRL+ALT+F1
  At the ESXi shell login with root and the password
  Run the following commands to show number of failed attempts:

  pam_tally2 --user root
  Run the following command to unlock the root account:

  pam_tally2 --user root --reset
  ```` 
* VMWARE - Lista ISO "atachadas" nas maquinas virtuais
  ```` 
  Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
  Connect-VIServer -Server <IP_VCENTER> -User <ADMINISRADOR_TOP>  -Password <SENHA_ADMIN_TOP>
  
  PS /root> $VMs = Get-VM
  PS /root> $Output = foreach ($VM in $VMs){ Get-CDDrive $VM | select Parent, IsoPath }
  PS /root> echo $Output                  #exibir na tela
  PS /root> $Output | Export-Csv          #mandar para csv
  ````
* VMWARE - Habilitar SNMP no Host Vmware
  ```` 
  esxcli system snmp set -r                                                            #reset na config
  esxcli system snmp set -c  <comunidade>                                              #comunidade do snmp, recomendado nao utilizar public
  esxcli system snmp set -p 161                                                        #PORTA
  esxcli system snmp set -C  ch@rles.xyz                                               #syscontact ou contato 
  esxcli system snmp set -L  "Em algum lugar no Data Center"                           # syslocation ou localizacao 
  esxcli network firewall ruleset set --ruleset-id snmp --allowed-all false            #configurcao do firewall 
  esxcli network firewall ruleset allowedip add --ruleset-id snmp --ip-address X.X.X.X # <IP OU RANGE do Servidor que fara a coleta>
  esxcli network firewall ruleset set --ruleset-id snmp --enabled true                 #configuracao do firewall 
  esxcli system snmp set --enable true                                                 #habilita servico
  /etc/init.d/snmpd restart                                                            #restart do servico
  ```` 
 * VMWARE - Configurar placa de rede via CLI no VCenter 6.x
   ```` 
   /opt/vmware/share/vami/vami_config_net
   ```` 
 * VMWARE - Habilitar sshd via CLI no VCenter 6.x
   ```` 
   systemctl unmask sshd
   systemctl start sshd
   systemctl enable sshd
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
* FGT - FortiGate Authentication timeout - Tempo de autenticacao do usuario conectado na vpn, por padrão 8hs, depois o usuario é derrubado.
  ````
  config vpn ssl settings
     set auth-timeout 42200  (12hs) 
  end 
  ````  
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
  - Fonte: https://forum.fortinet.com/tm.aspx?m=156688
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
* FGT - Debug registro do Fortigate nos servidores da FGT e atualização das licenças (versão melhorada do item anterior).
  ````
  diag debug reset
  diag debug flow filter clear
  diag debug flow show  function-name enable 
  diag debug flow show iprope enable 
  diag debug flow filter addr 173.243.138.68 
  diag debug flow filter addr 12.34.97.16 
  diag debug flow filter addr 208.91.112.53
  diag debug flow filter addr 96.45.33.88
  diag debug flow trace start 100
  diagnose debug application update -1
  diagnose debug enable 
  execute update-now 
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
  - Fonte: http://kb.fortinet.com/kb/viewContent.do?externalId=FD30618 
  ````
  diag test application dnsproxy 1
  diag test application dnsproxy ? # ver outros niveis de teste
  ````
* FGT - Falha na comunicação com DNS Filter Servers INDIPONIVEIS 
  ````
  config system fortiguard
  set sdns-server-ip 45.75.200.89
  end
  ````
* FGT - Forticloud falha de comunicação com o fortigate, não iniciando o tunel para gerenciamento. 
  ````
  LOG: Configuration of this device has not been initialized in FortiGate Cloud, please set Central Management to FortiGate Cloud from device and verify the management tunnel is up. And it may take a few seconds to initialize. 

  config system central-management
    set type fortiguard
  end
  ````
## Zabbix

* ZBX - “rascunho” basico de como usar a API do Zabbix com o curl do linux, objetivo de demonstrar o funcionamento.
  - Fonte: Maintenance [Zabbix Documentation 1.8] // host.get [Zabbix Documentation 5.0] // API [Zabbix Documentation 5.0] // Tutorial Zabbix API - Quickstart Guide [ Step by Step ]
  ````
  #Vou iniciar umas variaveis para facilitar a nossa vida :D (não é SH/BASH/KSH, as variaveis precisam ser substituidas manualmente)
  $HOST=<URLZABBIX>/api_jsonrpc.php 
  $USER=zbx_api
  $PASS=SDEDFRF

  #Login no zabbix e receber o KEY da conexão.
  
  curl --insecure   -i -X POST -H 'Content-type:application/json' -d '{"jsonrpc":"2.0","method":"user.login","params":{ "user”:”$USER ,”password”:”$PASS” },”auth":null,"id":0}' https://$HOST

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:00:29 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Content-Length: 68
  Content-Type: application/json
  
  {"jsonrpc":"2.0","result":"05d129888aff2d54563772d32c6b8e04","id":0}

  #Retorna todos os hosts do zabbix

  curl --insecure   -i -X POST -H 'Content-type:application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"output": ["hostid", "name"]},"auth":"05d129888aff2d54563772d32c6b8e04","id":0}' https://$HOST

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:02:28 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Transfer-Encoding: chunked
  Content-Type: application/json

  {"jsonrpc":"2.0","result":[{"hostid":"10084","name":"Zabbix server"}, #e tudo mais que tiver no zabbix .......

  #Retorna somente 1 maquina

  curl --insecure   -i -X POST -H 'Content-type:application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"filter": ["host", "teste"]},"auth":"05d129888aff2d54563772d32c6b8e04","id":0}' https://$HOST
  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:11:36 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Transfer-Encoding: chunked
  Content-Type: application/json

  {"jsonrpc":"2.0","result":[{"hostid":"10084","proxy_hostid":"0","host":"Zabbix server","status":"0","disable_unti

  #Recuperar o host ID, nome do host no campo host do inventario

  curl --insecure   -i -X POST -H 'Content-type:application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"output": ["hostid"], "filter": { "host": "db01" }  },"auth":"05d129888aff2d54563772d32c6b8e04","id":0}' https://$HOST 

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:38:22 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Content-Length: 54
  Content-Type: application/json

  {"jsonrpc":"2.0","result":[{"hostid":"10269"}],"id":0}

  #Recuperar o host ID, nome do host no campo host do inventario

  curl --insecure   -i -X POST -H 'Content-type:application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"output": ["hostid"], "filter": { "host": "db01l" }  },"auth":"05d129888aff2d54563772d32c6b8e04","id":0}' https://$HOST

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:38:22 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Content-Length: 54
  Content-Type: application/json

  {"jsonrpc":"2.0","result":[{"hostid":"10269"}],"id":0}
  
  #Recuperar o host ID, nome do "name" no campo host do inventario

  curl --pe:application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"output": ["hostid"], "filter": { "name": “SERVIDOR XXX” }  },"auth":"05d129888aff2d54563772d32c6b8e04","id":0}' https://$HOST

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 13:59:10 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Content-Length: 54
  Content-Type: application/json

  {"jsonrpc":"2.0","result":[{"hostid":"10372"}],"id":0}


  #Criar janela de manutenção, e ativar manutenção no host.  
  #Para data de inicio, é em segundo… então `date %s` e o retorno somar mais o tempo desejado. :D 
  #exemplo calculo do inicio e fim
  $date +%s
  1595524464 
  $echo $((1595513595 + 3600))
  1595517195
  $

  curl --insecure   -i -X POST -H 'Content-type:application/json' -d ' {"jsonrpc":"2.0", "method":"maintenance.create", "params":[{ "groupids":[], "hostids":["10425"], "name":"MANUTENCAO PROGRAMADA", "maintenance_type":"0", "description":"MANUTENCAO HOSTS XXXX", "active_since":"1595524464", "active_till":"1595517195", "timeperiods": [ { "timeperiod_type": 0, "start_date": "1595524464", "period": 600 } ] }], "auth":"05d129888aff2d54563772d32c6b8e04","id":3}' https://$HOST

  HTTP/1.1 200 OK
  Date: Thu, 23 Jul 2020 14:15:15 GMT
  Server: Apache/2.4.29
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Headers: Content-Type
  Access-Control-Allow-Methods: POST
  Access-Control-Max-Age: 1000
  Content-Length: 58
  Content-Type: application/json

  {"jsonrpc":"2.0","result":{"maintenanceids":["1"]},"id":3}


<h6>
Obs.: Maioria destes comandos foram utilizados para resolver problemas pontuais, a alguns são de muito, muito tempo atrás, não possuem nenhuma "boniteza" e organização nos mesmos. São mais como notas para não esquecimento :D 
</h6>
