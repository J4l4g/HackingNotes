#AD #ACL #DACL #SACL
 
> Las ACL son los permisos que se le asignan a usuarios y equipos para regular a que archivos y objetos pueden acceder
> 
> Existen dos tipos de ACL
> - Discretionary Access Control List (DACL): regula que entidades tienen acceso aun objeto
> - Sistem Acceses Control List (SACL): regula y registra los intentos de acceso a un acceso
> 
> Nos podemos aprovechar de las ACL para realizar movimiento lateral, escalda de privilegios y crear persistencia

# Enumeración ACL
Lo primero que debemos de realizar es importar el modulo de [[POWERVIEW]]
- Cuando tenemos un usuario valido creamos una variable que sera su nombre pasado a SID
	```shell
	$sid = Convert-Name-ToSid <user>
	```
	