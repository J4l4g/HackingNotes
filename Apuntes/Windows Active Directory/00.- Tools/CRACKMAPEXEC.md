*Esta herramienta nos permite evaluar diferentes campos de los sistemas dependiendo de la fase en la que nos encontremos del pentest, permitiéndonos por ejemplo hacer una enumeración de usuarios y contraseñas.*

###### Enumeración de políticas de contraseñas con credenciales validas
```bash
crackmapexec smb <ip> -u <user> -p <password> --pass-pol
```


| Opciones     | Usos                               | + Info |
| ------------ | ---------------------------------- | ------ |
| *-u*         | Usuario valido                     |        |
| *-p*         | Contraseña valida                  |        |
| *--pass-pol* | Enumerar políticas de contraseñaas |        |
