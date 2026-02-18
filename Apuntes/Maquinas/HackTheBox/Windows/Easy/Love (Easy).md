
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

Encontramos un panel de login de `Admin` así que vamos a buscar vulnerabilidades que pueda tener el sistema de `Voting System`, vamos a usar
