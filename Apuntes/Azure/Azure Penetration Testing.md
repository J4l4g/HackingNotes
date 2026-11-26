Azure Active Directory *(AAD)* es un servicio de gestión de identidades y accesos basado en la nube

Terminología
- *Tenant* -> Una instancia de Azure que representa una sola organización.
- *Azure AD Directory* -> Un directorio dedicado para cada *Tenanat*, esto es usado para realizar funciones de administración de identidades y acceso para los recursos.
- *Subscriptions* -> Se usa para pagar los servicios puede haber varios en un *Directory*.
- *Core Domain* -> Es el dominio principal `<tenenat>.onmicrosoft.com`, también estos nombres pueden ser nombres de dominio personalizados.

Arquitectura
- *Management Groups*
- *Subscriptions*
- *Resaource Groups*
- *Resources*
![[Pasted image 20261126170237.png]]

### Management Groups
Son usados para gestionar múltiples subscripciones, todas las subscripciones heredan las condiciones aplicadas a los *Managements groups*
Un *Mangemenent Group* puede estar ubicado por debajo de otro
Existe un único *Management Group* de nivel superior en cada directorio de Azure este se llama *Root Management Group*

### Subscriptions
Una subscripcion es una unidad logica de servicios de Azure que se vincula a una cuenta de Azure
