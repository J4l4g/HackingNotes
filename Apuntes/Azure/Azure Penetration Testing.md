Azure Active Directory *(AAD)* es un servicio de gestión de identidades y accesos basado en la nube

Terminología
- *Tenant* -> Una instancia de Azure que representa una sola organización.
- *Azure AD Directory* -> Un directorio dedicado para cada *Tenanat*, esto es usado para realizar funciones de administración de identidades y acceso para los recursos.
- *Subscriptions* -> Se usa para pagar los servicios puede haber varios en un *Directory*.
- *Core Domain* -> Es el dominio principal `<tenenat>.onmicrosoft.com`, también estos nombres pueden ser nombres de dominio personalizados.

## Arquitectura
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

### Groups and Resources
Los *Resources* son recursos que se pueden implementar en Azure, como maquinas virtuales, servicios de aplicaciones, cuestas de almacenamiento, etc. Estos recursos pueden ser almacenados en los *Resources Groups* que actúan como contenedores
Todos los recursos deben de estar dentro de un grupo y solo pueden pertenecer a ese grupo
Si se elimina un *Resource Group* todos los recursos de dentro de este se eliminan con el
Un *Resource Group* tiene su propia configuración de *Identity and Access Management* para proporcionar un acceso basado en roles. Un rol aplicado a un grupo se aplica a todos los recursos que haya dentro de este


## Roles de Azure RBAC
Hay mas de 120 roles integrados y además se pueden definir roles personalizados
Cuatro roles fundamentales

| Rol                       | Permisos                                                                            |
| ------------------------- | ----------------------------------------------------------------------------------- |
| Owner                     | Acceso completo a todos los recursos<br>Puede administrar accesos de otros usuarios |
| Contributor               | Acceso completo a todos los recursos<br>No puede administrar accesos                |
| Reader                    | Puede ver todos los recursos                                                        |
| User Access Administrator | Puede ver todos los recursos<br>Puede administrar el acceso de otros ususarios      |
![[Pasted image 20261126174541.png]]


### Azure Resource Manager (ARM)
Es el servicio de administración e implementación independiente del cliente para Azure, que se utiliza para la administración del ciclo de vida (creación, actualización y eliminación) y el control de acceso a los recursos
Las plantillas *ARM* se pueden utilizar para reimplementar de forma coherente y definida por dependencias de los recursos
![[Pasted image 20261126180251.png]]


# AZURE Kill Chain
![[Pasted image 20261126182206.png]]

## Reconocimiento
Conociendo únicamente el nombre de dominio o un email de la organización objetivo
- `retailcorp`
Podremos extraer información relevante a la hora de realizar las pruebas
- Si la organización usa *Azure tenant*
- *Tenant ID*
- *Tenant name*
- Tipo de autenticación *Federation o no*
- *Domains*
- Servicios de Azure usados por la organización
- Obtener *IDs* de los correos electrónicos
#### Descubrir cómo gestiona la autenticación el dominio de una organización
```URL
https://login.microsoftonline.com/getuserrealm.srf?login=USERNAME@retailcorp.onmicrosoft.com&xml=1
```

Esta técnica se llama *User Realm Discovery*
- El endpoint `getuserrealm.srf` recibe una dirección de correo electrónico a *UPN (User Principal Name* y devuelve información sobre el dominio y el mecanismo de autenticación asociado
	- `login` campo donde se recibe *UPN* o *mail*
	- `xml=1` solicita respuestas en formato *XML*

| Campo                 | Informacion                                               |
| --------------------- | --------------------------------------------------------- |
| `Success`             | Si la consulta se ha procesado correctamente              |
| `State`               | Estado interno de la resolucion del realm                 |
| `UserState`           | Estado asociado a la identidad asociada                   |
| `NameSpaceType`       | Tipo de auntenticacion del dominio                        |
| `DomainName`          | Dominio asociado al realm                                 |
| `IsFederatedNS`       | Si el namespace utiliza federacion                        |
| `FederationBrandName` | Nombre o marca de la organizacion federada                |
| `AuthURL`             | URL del provedor de autenticacion del propveedor federado |
| `CloudInstanceName`   | Intancia de nube de Microsoft utilizada                   |
#### Obtener el ID de Tenant
```URL
https://login.microsoftonline.com/retailcorp.onmicrosoft.com/.well-known/openid-configuration
```

Esta técnica se conoce como *OpenID Connect Discovery Endpoint*, permite descubrir como esta configurado el servicio de autenticación y que endpoint puedes usar para interactuar con el

Esta URL devuelve en formato JSON toda la información publica sobre la configuración *OpenID Connect (OIDC)* del tenant

#### Enumeracion de usuarios
```URL
https://login.microsoftonline.com)/common/GetCredentialType
```

Se hace una consulta a un endpoint interno de *Microsoft Entra ID* que permite consultar como debe procesarse el inicio de sesión de un usuario y que tipo de credenciales o proveedor de identidad utiliza

La forma mas comoda de usarlo es traves de curl
```shell
curl -s -X POST \
"https://login.microsoftonline.com/common/GetCredentialType" \
-H "Content-Type: application/json" \
-d '{
  "Username": "usuario@retailcorp.com"
}'
```

Podemos obtener tres tipos de respuestas
- `If`