# Evasion de politicas de ejecucion PowerShell
```shell
powershell -ep bypass
```

# Iniciar PowerView
```shell
. .PowerView.ps1
```

# Enumeracion
## Usuarios
```shell
Get-NetUser | select cn
```

```shell
net users
```

### Informacion especifica sobre un usuario
```shell
net user <nombre>
```

## Grupos
```shell
Get-NetGroup
```

### Grupos que contengan la palabra *admin*
```shell
Get-NetGroup -GroupName *admin*
```

## Usuario actual
### Privielgios
```shell
whoami /priv
```

### Grupos a los que pertenece
```shell
whoami /groups
```