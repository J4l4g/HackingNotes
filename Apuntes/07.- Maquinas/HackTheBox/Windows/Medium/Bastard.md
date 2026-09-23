
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.61.47 -oG allPorts
```

Ahora haremos el reconocimiento mas exhaustivo sobre los puertos encontrados `80`, `135` y `49154` usando [[NMAP]]
```shell
nmap -p80,135,49154 -sCV -vvv 10.129.61.47 -oN targeted
```

