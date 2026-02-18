#AD #ACL #DACL #SACL
 
> Las ACL son los permisos que se le asignan a usuarios y equipos para regular a que archivos y objetos pueden acceder
> 
> Existen dos tipos de ACL
> - Discretionary Access Control List (DACL): regula que entidades tienen acceso aun objeto
> - Sistem Acceses Control List (SACL): regula y registra los intentos de acceso a un acceso
> 
> Nos podemos aprovechar de las ACL para realizar movimiento lateral, escalda de privilegios y crear persistencia

# Enumeración ACL
## Con PowerVi
Lo primero que debemos de realizar es importar el modulo de [[POWERVIEW]]
- Cuando tenemos un usuario valido creamos una variable que será convertir su nombre de usuario en su SID

```shell
$sid = Convert-Name-ToSid <user>
```

- Buscaremos todos los objetos sobre los que nuestro usuario tiene permisos

```shell
GetDomainObjectACL -Identity * | ? {$-.SequrityIdentifier -eq $sid}
```

Con el `GUID` que nos devuelve la salida del comando podemos buscar en Google que valor le corresponde, como por ejemplo `Forzar Cambios de Contraseñas`.
En vez de poder buscarlo en Google podemos usar la opción `-ResolveGUIDs`

```shell
GetDomainObjectACL -ResolveGUIDs -Identity * | ? {$-.SequrityIdentifier -eq $sid}
```

