*Esta herramienta nos permite interactuar directamente con los servicios RPC de Windows directamente muchas veces sin la necesidad de tener credenciales validas, permitiéndonos así hacer diferentes enumeraciones en le host.*

###### Acceso con NULL-SESSION
```bash
rpcclient -U "" -N <ip>
```


| Opciones | Usos                                     | + Info                                                            |
| -------- | ---------------------------------------- | ----------------------------------------------------------------- |
| *-U*     | Usuario                                  | Si el `""` se queda vacío indica que quieres acceder sin usuarios |
| *-N*     | Indica que vamos a usar una NULL-SESSION |                                                                   |

###### Verificar el correcto acceso al host
`QUERYDOMAININFO` 
###### Enumeración de políticas de contraseñas
`GETDOMPWINFO`
