
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

