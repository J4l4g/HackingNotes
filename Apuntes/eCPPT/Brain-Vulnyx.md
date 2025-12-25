

### Enumeración de redes
*ARP-SCAN*
`arp-scan -I eth0 --localnet`


### 192.168.1.93
#### Enumeración de puertos y servicios
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 192.168.1.93 -oG allPorts`

