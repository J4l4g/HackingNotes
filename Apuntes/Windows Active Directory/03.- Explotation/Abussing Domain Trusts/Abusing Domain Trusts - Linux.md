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
```shell-session
<dominio.secundarios>/<usuario.admin>@<IP> -just-dc-user <dominio>/krbtgt
```
```