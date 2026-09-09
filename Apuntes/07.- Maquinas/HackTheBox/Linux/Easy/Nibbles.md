```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.96.84 -oG allPorts
```

```shell
nmap -p22,80 -sCV -vvv 10.129.96.84 -oN targeted 
```

Accederemos al puerto `80` a través del navegador web para ver que aloja este
![[Pasted image 20260909205354.png]]

Nos encontramos con un *Hello World!* vamos a usar [[WHATWEB]] en busqueda de las tecnoligas que usa
```shell
whatweb http://10.129.96.84/
```

Sin encontrar nada relevante echaremos de nuevo un ojo a la web en búsqueda de cualquier indicio que nos pueda ayudar.
Vemos el código fuente de la pagina y encontramos que hay un directorio llamado `/nibbleblog/` 
![[Pasted image 20260909205610.png]]

Accederemos a el
![[Pasted image 20260909205649.png]]

Encontramos un directorio semejante a una web al estilo de un blog en el que se habla de últimos post, paginas, etc. Este se llama *Nibbleblog*
Vamos a hacer una busqueda en internet para conocer mas sobre este, vemos que este se trata de un *CMS* 
Conociendo este directorio vamos a hacer fuzzing sobre el
```shell
ffuf -c -fc 404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://10.129.96.84/nibbleblog/FUZZ
```

Encontramos directorios como `admin`, `content`, `lenguages`, `themes` y `plugins`
Navegando por estas rutas encontramos 
