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
* ZIMBRA - Mostrar as contador de mensagens e pastas de um usuario via CLI.
  ````
  zmmailbox -z -m <usuario@domino> gaf | more
  ````
* ZIMBRA - Mostrar contador <b>aproximado<b> de mensagem total da caixa de um usuario. <b<Aproximado</b< porque, soma shared-folders e outros itens junto, precisa melhorar o filtro para somar somente mensagens.
  ````
  total=0; for a in `zmmailbox -z -m  gaf <usuario@domino> | egrep "mess|unkn" | grep -vi contac  |  awk '{ print $4 }' | egrep -o "[0-9]+" ;`; do total=$(( total + a )); done; echo "$total"
  ````
<hr>

## Linux

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
   esxcli system maintenanceMode set –enable true  #procedimento deve ser realizado em modo manutenção.             
   esxcfg-nics -l                                  #exibe os drivers que estão sendo utilizados
   esxcfg-module -d ntg3                           #desabilita o driver ntg3
   esxcfg-module -e tg3                            #habilita o driver tg3
   init 6                                          #reboot
   esxcli system maintenanceMode set --enable false  #sair do modo manutenção     
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

<h6>
Obs.: Maioria destes comandos foram utilizados para resolver problemas pontuais, a alguns são de muito, muito tempo atrás, não possuem nenhuma "boniteza" e organização nos mesmos. São mais como notas para não esquecimento :D 
</h6>
