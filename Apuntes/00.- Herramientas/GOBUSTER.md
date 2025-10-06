#Enumeration #WebEnumeration 

| OPCIONES          | FUNCION                              | +Info                                                                   |
| ----------------- | ------------------------------------ | ----------------------------------------------------------------------- |
| *dir*             | Directorios                          |                                                                         |
| *-u*              | URL                                  |                                                                         |
| *-w*              | Wordlist                             | /usr/share/seclists/Discovery/Web-Content/common.txt                    |
| *-x*              | Extensiones                          | html, php, txt, py                                                      |
| *-k*              | No SSL                               | Si hay puerto 80 y 443 evitar el SSL                                    |
| *-d*              | dns                                  | Modo dns `dns -d <nombred_dominio.com>`                                 |
| *vhost*           | Enumeración Vhost                    |                                                                         |
| *-t*              | Numero de hilos                      | Numero de tareas en paralelo                                            |
| *--add-slash*     | Para forzar búsquedas con / al final |                                                                         |
| *-b*              | Blacklist códigos de estado          |                                                                         |
| *-s*              | Codigos de estado que nos interesa   |                                                                         |
| *vhost*           | Búsqueda subdominios                 |                                                                         |
| *--append-domain* | Enumeracion de subdominios + .       | Se usa para enumerar subdominios y añadirlo antes del dominio principal |
Podemos hacer un grep para eliminar los códigos de error `403` con `grep -v "403"`
