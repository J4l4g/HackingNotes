## Reconocimiento
Empezaremos haciendo uso de [[NMAP]] para hacer un reconocimiento inicial
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

Nos encontramos abiertos los puertos `22/ssh` `80/HTTP`
Navegamos al puerto `80` via web y nso encvon