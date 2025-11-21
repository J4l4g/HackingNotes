
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

```

