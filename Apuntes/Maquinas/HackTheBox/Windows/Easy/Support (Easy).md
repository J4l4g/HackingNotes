

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

Abrimo el domcumento de los `host` de Windows copn el bloc de notas como administrador, este fichero de configuracion se encuentra en `C:\Windows\System32\drivers\etc\hosts`