#Enumeration #RedEnumeration

| OPCIONES       | FUNCION                          | +Info     |
| -------------- | -------------------------------- | --------- |
| *-I*           | Interfaz de escaneo              |           |
| *--localnet*   | Indicar que red vamos a escanear | Red local |
| *--ignoredups* | Ignorar duplicados               |           |

Para escanear equipos en la red local usaremos
``` bash
arp-scan -I <Interfaz> --localnet
```
