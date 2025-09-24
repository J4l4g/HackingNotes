Empezaremos realizando un escaneo con [[NMAP]] en el cual encontramos abierto el purto `22/tcp` y el puerto `500/udp`
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`
`nmap -sU -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

En el puerto `500` hay un servicio `ISAKMP`

