
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.58.162 -oG allPorts
```

Encontramos los puertos `135`, `139`, `445`, `49152`, `49153`, `49154`, `49155`, `49156`, `49157` sobre los que haremos un escaneo exhaustivo con [[NMAP]]
```shell
nmap -p135,139,445,49152,49153,49154,49155,49156,49157 -sCV 10.129.58.162 -oN targeted
```

Vemos que es un Windows 7, enumeraremos el servicio *SMB* 
