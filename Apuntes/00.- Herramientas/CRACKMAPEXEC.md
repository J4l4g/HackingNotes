#Enumeration #RecursosCompartidos #SMBEnumeration 

*Esta herramienta nos permite evaluar diferentes campos de los sistemas dependiendo de la fase en la que nos encontremos del pentest, permitiéndonos por ejemplo hacer una enumeración de usuarios y contraseñas.*
###### Enumeración de políticas de contraseñas con credenciales validas
```bash
crackmapexec smb <ip> -u <user> -p <password> --pass-pol
```

| OPCIONES     | FUNCION                            | +Info                        |
| ------------ | ---------------------------------- | ---------------------------- |
| *protocolo*  | Enumerar Protocolo                 | Por ejemplo smb              |
| *-u*         | Usuario                            | Si no lo conocemos dejar ' ' |
| *-p*         | Password o Wordlist                | Si no lo conocemos dejar ' ' |
| *-k*         | Autentica por kerberos             |                              |
| *--users*    | Enumerar usuarios                  |                              |
| *--shares*   | Enumerar ficheros compartidos      |                              |
| *--pass-pol* | Enumerar políticas de contraseñaas |                              |

