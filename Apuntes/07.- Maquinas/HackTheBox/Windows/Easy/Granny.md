#CPTS 

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.95.234 -oG allPorts
```

Encontramos abierto el puerto `80` así que haremos un reconocimiento tan exhaustivo sobre este
```shell
nmap -p80 -sCV -vvv 10.129.95.234 -oN targeted
```

