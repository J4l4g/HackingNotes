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
Una suscripción es una unidad lógica de servicios de Azure que se vincula a una cuenta de Azure
Las subscripciones son un limite y/o control de acceso en un *Azure AD Directory*
Cada *Azure AD Directory* pude tener varias subscripciones pero cada una de estas solo puede confiar en un único directorio
Un rol de Azure aplicado a nivel de suscripción se aplica a todos los recursos dentro de esta

### Resource Groups
Los *Resources* son recursos que se pueden implementar en Azure, como maquinas virtuales, servicios de aplicaciones, cuestas de almacenamiento, etc. Estos recursos pueden ser almacenados en los *Resources Groups* que actúan como contenedores
Todos los recursos deben de estar dentro de un grupo y solo pueden pertenecer a ese grupo
Si se elimina un *Resource Group* todos los recursos de dentro de este se eliminan con el
Un *resource Group* tiene su propia configuración de *identity and Access Management* para proporcionar un acee

