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

## Grupos
```shell
Get-NetGroup
```

### Grupos que contengan la palabra *admin*
```shell
Get-NetGroup -GroupName *admin*
```
