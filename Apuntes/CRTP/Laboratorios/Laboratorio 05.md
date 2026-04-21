
## Tareas
- [ ] Explotar un servicio en *dccorp-student97* y elevar los privilegios a administrador local
- [ ] Identificar una maquina en el dominio donde student97 tenga acceso administrativo local
- [ ] Utilizando los privilegios de un usuario en Jenkins *172.16.3.11:8080* obtener privilegios de administrador en *172.16.3.11*, el servidor *dcorp-ci*

**Flag 5 Servicio utilizado indebidamente en la maquina que permite la escalada de privilegios locales**

**Flag 6 Script utilizado para la búsqueda de privilegios mediante PowerShell Remoting**

**Flag 7 Usuario de Jenkins utilizado para acceder a la consola web de esta**

**Flag 8 Usuario de dominio utilizado para ejecutar Jenkins en dcorp-ci**

## Enumeración
Primero deberemos de ejecutar *InviShell* para poder eludir las detecciones de PowerShell y poder ejecutar este de forma mas sigilosa
```shell
. .\InviShell\RunWithRegistryNonAdmin.bat  
```

Y después podremos ejecutar *PowerUp*
```shell
. .\PowerUp.ps1 
```

### Enumerar servicios vulnerables
Este tipo de enumeración nos va a permitir encontrar servicios vulnerables que tengan activos los parámetros `CanRestart: True`, `Modifiable Services` y `Unquoted Service Paths`, con funciones de abuso relacionadas para explotarlas
