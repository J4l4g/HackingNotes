
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.61.155 -oG allPorts
```

Encontramos los puertos `22` y `80` abiertos, haremos un escaneo mas exhaustivo de estos puertos
```shell
nmap -p22,80 -sCV -vvv 10.129.61.155 -oN targeted
```


