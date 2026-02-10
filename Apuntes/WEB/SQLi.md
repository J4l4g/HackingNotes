## Payloads

### Identificación de versión y motor de base de datos
#### ORACLE
Identificar el numero de columnas
`' order by 5-- -`, debemos de ir cambiando el número hasta que veamos que cambia algo y ya no nos devuelve un error

Una vez ya hemos identificado las columnas debemos probar con un
`' union select NULL,NULL-- -`, no mostrarnos nada nos esta dando una piesta de que puede ser una base de datos ORACLE para ello deberemos refenciar a una tyabla

