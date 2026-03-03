>Es una técnica de movimiento lateral/escalada de privilegios en entornos de Active Directory que explota una configuración débil en las cuentas de usuario. A diferencia de [[Kerberoasting]], este ataque no se dirige a cuentas con SPN si no a cuentas de usuario que tengan habilitada la opción "Do not require Kerberos preauthentication" .

# Ejecución del ataque

Este ataque se puede realizar siempre que:
- Un usuario válido en el dominio (sin necesidad de privilegios especiales).
- Conectividad con el Controlador de Dominio.
- Capacidad para ejecutar herramientas que puedan solicitar tickets Kerberos.
# Herramientas

Las herramientas necesarias para poder desarrollar este ataque son:
- [[IMPACKET]] con  `GetNPUsers`, desde Linux
- [[RUBEUS]] (con el argumento `/asreproast`) o [[POWERVIEW]] (con el comando `Get-DomainUser -PreauthNotRequired`), desde Windows

# Requisitos previos
