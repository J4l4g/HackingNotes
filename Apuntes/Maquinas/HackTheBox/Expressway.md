Empezaremos realizando un escaneo con [[NMAP]] en el cual encontramos abierto el purto `22/tcp` y el puerto `500/udp`
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`
`nmap -sU -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

En el puerto `500` hay un servicio `ISAKMP Internet Security Association and Key Management Protocol`
Usaremos [[IKE-SCAN]] es una utilidad de línea de comandos para descubrir, identificar (fingerprint) y recopilar información de hosts que hablan IKE (Internet Key Exchange)

Ejecutaremos el siguiente comando:
`ike-scan --aggressive --pskcrack=psk.params`
`--aggresive` -> Envía identidades y parámetros en los primeros mensajes, sin esperar un handshake completo.
`--pskcrack` -> Captura el handshake en Aggressive Mode y lo guarda en `psk.params`

Usaremos [[PSK-CRACK]] 
