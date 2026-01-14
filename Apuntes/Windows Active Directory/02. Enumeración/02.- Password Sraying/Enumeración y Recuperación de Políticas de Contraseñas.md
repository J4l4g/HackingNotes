## Desde Linux
### Con credenciales
Teniendo credenciales validas podemos usar [[CRACKMAPEXEC]], para así enumerar las políticas de contraseñas y poder dirigir mejor nuestros ataques.

### Sin credenciales
#### RPCCLIENT
Sin tener una credenciales validas podemos usar [[RPCCLIENT]], y enumerar estas políticas de contraseñas usando NULL-SESSION.

Una vez habiendo podido acceder con una sesión comprobaremos si hemos accedido correctamente al host:
`QUERYDOMAININFO`

En caso de que hayamos accedido correctamente lo que tendremos que hacer será identificar las políticas de contraseñas, en este caso usaremos el comando:
`GETDOMPWINFO`

#### ENUM4LINUX
Sin tener credenciales validas también podemos enumerar con [[ENUM4LINUX]]

En caso de querer usar algo mas nuevo y actualizado y con muchos parámetros mas para poder modificar podemos usar

#### ENUM4LINUX-NG
Esta versión tiene funciones adicionales al normal [[ENUM4LINUX-NG]]


## Desde Windows
### Sin credenciales
Usaremos los comando `NET`.
#### Establecer una Null-Session
`net use \\DC01\ipc$ "" /u:""`

#### Errores
Los errores que nos proporciona el comando anterior nos puedan dar muchas pistas sobre los usuarios.
- *1331* - Cuenta desactivada.
- *1326* - Contraseña incorrecta.
- *1909* - Cuenta bloqueada (Política de contraseña)
