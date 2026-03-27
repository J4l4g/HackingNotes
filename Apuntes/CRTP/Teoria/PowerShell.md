#### Listar servicios en ejecución
```powershell
Get-Service | Where-Object { $_.Status -eq "Running" }
```

