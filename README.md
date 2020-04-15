# Dicas diversas 
Dicas diversas sobre comandos linux, MACOS, Fortinet, Zimbra, e outros.... 

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
* LINUX - A partir do redhat/centos 7 não temos mais netstat 
  <h6> Referencia: https://www.nixpal.com/netstat-ss-and-rhel-7-centos/ </h6>
  
  | Command | Description |
  | --- | --- |
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
  | s -o state fin-wait-1 \( sport = :http or sport = :https \) dst <IP> | | lista todas as conexão com porta origem 80 OU 443, do statdo fin-wait-1 e com destino <IP> |  
 


<h6>
Obs.: Maioria destes comandos foram utilizados para resolver problemas pontuais, a alguns são de muito, muito tempo atrás, não possuem nenhuma "boniteza" e organização nos mesmos. São mais como notas para não esquecimento :D 
</h6>
