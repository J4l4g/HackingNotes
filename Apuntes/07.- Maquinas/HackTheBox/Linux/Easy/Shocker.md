```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.55.91 -oG allPorts
```

```shell
nmap -p80,2222 -sCV -vvv 10.129.55.91 -oN targeted
```

Encontramos abiertos los puertos `80` con una *web* y `2222` con un *ssh*
Navegaremos a la web para obtener información sobre ella
Primero veremos que alberga la web en busqueda de informacion relevante