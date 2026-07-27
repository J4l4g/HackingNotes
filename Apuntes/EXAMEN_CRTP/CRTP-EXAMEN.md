### Quien soy
- [ ] `whoami`
- [ ] `whoami /all`

### Cargar PowerView
- [ ] `. .\PowerView.ps1`
- [ ] `Import-Module PowerView.ps1`
- [ ] `powershell -ExecutionPolicy Bypass -Command ". .\PowerView.ps1"`

### Enumerar recursos compartidos
- [ ] `Invoke-ShareFinder`

### Leer recurso compartido
- [ ] `dir \\tech-dc.tech.corp\maintance`
- [ ] `copy \\tech-dc.tech.corp\maintance\* .`

### CMD como AdminLocal
- [ ] `runas.exe /user:studentadmin cmd`
- [ ] `whoami`
- [ ] `hostname`

### Añadir usuario al grupo Administratorsç
- [ ] `net localgroup Administrators TECH\studentuser /add`
- [ ] `net localgroup Administrators`







- [ ] .\SharpHound.exe -c All,GPOLocalGroup,LoggedOn --domain tech.corp
