
```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.16.146 -oG allPorts
```

```shell
 [*] IP Address: 10.129.16.146
 [*] Open ports: 22,80,443,3552
```

```shell
nmap -p22,80,443,3552 -sCV -vvv 10.129.16.146 -oN targeted
```

En la web del puerto 80 no encontramos anda relevante pero vemos que hay una web en el puerto 3552.
En este puerto se encuentra un servicio Arcane con la versión 1.13.0 la cual al buscarla en internet tiene varios CVE

