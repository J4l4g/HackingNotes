#Enumeration #RedEnumeration 

| OPCION          | FUNCION                              | +Info                                           |
| --------------- | ------------------------------------ | ----------------------------------------------- |
| *-sn*           | No hace escaneo de puertos           | Sirve para escanear el rango                    |
| *-p-*           | Todos los puertos tcp                |                                                 |
| *-sS*           | Escaneo sigiloso                     |                                                 |
| *-sC*           | Escaneo de scripts                   |                                                 |
| *-sV*           | Escaneo de versiones                 |                                                 |
| *-F*            | No puertos filtrados                 |                                                 |
| *--min-rate X*  | Minimo de paquetes enviados          | Cuanto mas bajo vas lento y menos satura la red |
| *-n*            | No usar resolucion DNS               |                                                 |
| *-vvv*          | Verbose                              |                                                 |
| *-Pn*           | No utilizar ping                     |                                                 |
| *-oN*           | Guardar en fichero formato nmap      |                                                 |
| *-A*            | Todas las opciones                   |                                                 |
| *-P*            | Rango de puertos                     |                                                 |
| *-oG*           | Guardar en fichero formato grepeable | Permite usar extractPorts                       |
| *-iL*           | Fichero con direcciones IP           |                                                 |
| *-sT*           | Escaneo puertos TCP                  |                                                 |
| *-sU*           | Escaneo puertos UDP                  |                                                 |
| *--top-ports X* | Escanear los X pueros mas comunes    | Aconsejable usar 200                            |
| *--open*        | Busca los puertos abiertos           |                                                 |
| *--script vuln* | Busca scripts de vulnerabilidades    |                                                 |
## Evasión de Firewalls

| OPCION          | FUNCION                                                        | +Info                  |
| --------------- | -------------------------------------------------------------- | ---------------------- |
| *-f*            | Fragmentacion de paquetes                                      |                        |
| *--mtu*         | Unidad de transmision maxima                                   | Valores múltiplos de 8 |
| *-D*            | Sustituir el origen de la dirección IP que envía el paquete    |                        |
| *--source-port* | Enviar los paquetes por un puerto especifico de nuestro equipo |                        |
| *--data-length* | Modificar la longitud de los paquetes                          |                        |
| *--spoof-mac*   | Modificar dirección Mac para el envió de paquetes              |                        |

## Scripts NMAP

Se encuentran en `/usr/share/nmap/scripts`
Las categorias se pueden buscar con: `locate.nse | xargs grep "categories" | grep -oP '".*?"' | sort -u` se pueden conbinar categorias

| OPCION     | FUNCION            | +Info                                         |
| ---------- | ------------------ | --------------------------------------------- |
| *--script* | Especificar script | --script="categoria/conbinacion de categorias |

