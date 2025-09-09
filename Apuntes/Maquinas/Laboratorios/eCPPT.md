#### Maquina 1: 192.168.1.23 

Hacemos un primer reconocimiento con [[NMAP]]
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 192.168.1.23 -oG allPorts`

Obtendremos los siguientes puertos:
```js
[*] IP Address: 192.168.1.23
[*] Open ports: 22,80
```

Hacemos un escaneo mas exhaustivo sobre estos dos puertos con [[NMAP]]
`nmap -p22,80 -sCV 192.168.1.23 -oN targeted`

Obtendremos la siguiente información de los puertos:
```js
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).


```