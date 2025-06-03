Hacemos un reconocimiento global con Nmap
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.10.11.47 -oG AllPorts`
 Hacemos reconocimiento de los puertos abiertos
 `nmap -p 22,80 --open -sS -sCV --min-rate 5000 -vvv -Pn -n 10.10.11.47 -oN PortsScan`

Agregamos al fichero /etc/hosts la IP y la URL para poder visualizarla
![[Pasted image 20241212181802.png]]

