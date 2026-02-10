*Esta herramienta nos permite hacer diferentes enumeración en los servicios LDAD, como puede ser por ejemplo enumeración de políticas de contraseñas*

```bash
ldapsearch -H <ip> -x -b "DC=<Nombre_dominio>,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

| Opciones | Usos                                                                     | + Info            |
| -------- | ------------------------------------------------------------------------ | ----------------- |
| *-H*     | IP maquina victima                                                       |                   |
| *-x*     | Usar autenticación simple                                                |                   |
| *-b*     | Punto desde donde empezar a buscar en este caso el dominio proporcionado |                   |
| *-s*     | Especificar el scope por defecto sub                                     |                   |
| *-D*     | Usuario de autenticación con el servidor                                 |                   |
| *-w*     | Credenciales/Passwd                                                      |                   |
