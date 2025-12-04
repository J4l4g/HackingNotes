Maquina atacante IP: *10.0.3.15*

### Enumeración de redes
*ARP-SCAN*
`arp-scan -I eth0 --localnet`

Encontramos la maquina en la red con IP `10.0.3.2`

## 10.0.3.2
### Reconocimiento de red
*NMAP*
`nmap -p- --open --min-rate 5000 -sS -Pn -n -vvv 10.0.3.2 -oG allPorts`

```ad-note
22/tcp   open  ssh           
135/tcp  open  msrpc         
445/tcp  open  microsoft-ds  
3389/tcp open  ms-wbt-server 
```

