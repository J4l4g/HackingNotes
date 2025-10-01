 j.fleischman / J0elTHEM4n1990!
## Reconocimiento
Empezamos realizando un reconocimiento con [[RUSTSCAN]]
`rustscan -a $IP --ulimit 1000 -r 1-65535 -- -A -sCV -o portScan`

Encontramos los siguientes puertos mas relevantes:
```js
53/DNS
88/Kerberos
389/LDAP
445/SMB
5985/WinRM
```

Miramos con las credenciales que nos han aportado que hay en el SMB
`netexec smb $IP -u "j.fleischman" -p 'J0elTHEM4n1990!' --shares`

Vemos que tenemos acceso con lectura y escritura al grupo de IT, accedemos a esta carpeta con
`smbclient '//10.10.11.69/IT' -U 'j.fleischman%J0elTHEM4n1990!'`

En esta carpeta nos encontramos un PDF que nos lo traemos a nuestra maquina para ver que contiene, el PDF contiene CVE de diversas maquinas, encontyramos que hay uin CVE-2025-24071 que permi


