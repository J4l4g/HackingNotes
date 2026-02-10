*Esta herramienta nos permite hacer enumeraciones de las maquinas victimas directamente desde nuestra maquina Linux pudiendo utilizar o no credenciales validas. Es igual que el normal solo que tiene funcionalidades mejoradas, obteniendo una salida mas clara.*

###### Obtener política de contraseñas
```bash
enum4linux-ng -P <ip> -oA <archivo_salida>
```

| Opciones | Usos                                                            | + Info |
| -------- | --------------------------------------------------------------- | ------ |
| *-P*     | Obtener información de la política de contraseñas               |        |
| *-oA*    | Archivo por el que queremos que se guarde la salida del comando |        |
