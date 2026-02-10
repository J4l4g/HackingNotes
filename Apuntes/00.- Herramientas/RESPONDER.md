*Esta herramienta que se encarga de explotar los protocolos LLMNR, NBT-NS y MDNS capturando las credenciales utilizadas para la autenticación.*

###### Configuración por defecto
```bash
sudo responder -I ens224 
```

| Opciones | Usos                                             | + Info                      |
| -------- | ------------------------------------------------ | --------------------------- |
| *-A*     | Analisis ver solicitudes NBT-NS, BROWSER y LLMNR | No contamina las respuestas |
| *-w*     | Inicia servidor proxy WPAD no autorizado         |                             |
| *-f*     | Identifica versión y SO del host remoto          |                             |
| *-I*     | Interfaz por la que escanear                     |                             |
