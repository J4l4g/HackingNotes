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

AHora para explotar la vulnerabilidad usaremos el script llamado `zzz_explo`