
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.58.185 -oG allPorts
```

Encontramos abiertos los puertos `21` y `80` haremos un reconocimiento mas exhaustivo de esos puertos con [[NMAP]]
```shell
nmap -p21,80 -sCV 10.129.58.185 -oN targeted 
```

![[Pasted image 20260916161706.png]]

Descubrimos que esta habilitado el acceso como *Anonymous* a través de *FTP*, así que accederemos usando este usuario al servicio

