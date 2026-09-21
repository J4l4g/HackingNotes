#CPTS 

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.95.234 -oG allPorts
```

Encontramos abierto el puerto `80` así que haremos un reconocimiento tan exhaustivo sobre este
```shell
nmap -p80 -sCV -vvv 10.129.95.234 -oN targeted
```

Obtendremos información sobre las tecnologías usadas por la web
```shell
whatweb http://10.129.95.234
```

Con el [[NMAP]] anteriormente usada vemos que hay un *WebDAV* que es un protocolo el cual nos permite guardar, editar, mover y compartir archivos en un servidor web.



