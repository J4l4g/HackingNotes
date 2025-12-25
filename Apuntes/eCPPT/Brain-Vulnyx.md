

### Enumeración de redes
*ARP-SCAN*
`arp-scan -I eth0 --localnet`


### 192.168.1.93
#### Enumeración de puertos y servicios
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 192.168.1.93 -oG allPorts`

```ad-info
PORT   STATE SERVICE
22/tcp open  ssh    
80/tcp open  http   
```


`nmap -p22,80 -sCV 192.168.1.93 -oN targeted`

#### Reconocimiento
### 80 http
Al navegar a la URL nos encontramos una pagina en la que se nos listan los rervicios que estan corriendo actualmente en la maquina victima, haremos una enumeracion de directorios con [[GOBUSTER]]
`gobuster dir -u http://192.168.1.93/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,html,txt`

Encontramos la ruta a `index.php`
Que es la misma ruta en la que nos hallamos al navegar a la pagina web

Como es un `.php` haremos fuzzing con [[WFUZZ]] en búsqueda de parametros de inyeccion despues del 