1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn <IP> -oN <NombreFichero>

2. Obtenemos un puerto abierto junto con su versión:
![[Pasted image 20241117172656.png]]

3. Buscamos en Internet algún exploit para la versión nombrada y lo descargamos:
`ftp vsftpd 2.3.4 exploit github`

4. Usamos el exploit el cual nos dará acceso a la maquina:
![[Pasted image 20241117172640.png]]
