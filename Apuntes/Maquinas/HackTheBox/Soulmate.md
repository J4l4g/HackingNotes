## Reconocimiento
Empezaremos haciendo uso de [[NMAP]] para hacer un reconocimiento inicial
`nmap -sSCV -p- --open --min-rate 5000 -n -Pn -vvv <IP>`

Nos encontramos abiertos los puertos `22/ssh` `80/HTTP`
Navegamos al puerto `80` vía web y nos encontramos una web `soulmate.htb` que la añadiremos al `/etc/hosts` junto con la IP correspondiente de la maquina, volvemos a recargar el navegador y ya nos carga la web.

Haremos fuzzing sobre los directorios con [[GOBUSTER]] `gobuster -u http://soulmate.htp` una vez hemos hecho esto y no hemos encontrado nada relevante

Haremos fuzzing de subdominios con [[FFUF]] `ffuf -u http://soulmate.htb -H "Host: FUZZ.soulmate.htb" \ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 4`
Encontramos un dominio llamdo `ftp.soulmate.htb` asi que navegamos a e