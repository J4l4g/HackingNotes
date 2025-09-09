#### Maquina 1: 192.168.1.23 

Hacemos un primer reconocimiento con [[NMAP]]
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 192.168.1.23 -oG allPorts`

Obtendremos los siguientes puertos:
```js
[*] IP Address: 192.168.1.23
[*] Open ports: 22,80
```

Hacemos un escaneo mas exhaustivo sobre estos dos puertos con [[NMAP]]
