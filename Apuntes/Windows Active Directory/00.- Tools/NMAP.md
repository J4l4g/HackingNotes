*Esta herramienta nos permite hacer un scaneo mas exaustivo de las maquinas de la red pudiendo devolvernos los puertos y servicios, las versiones de estos activos, vulnerabilidades conocidas, etc.*

```bash
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv <IP> -oG <archivo_salida>
```

```bash
nmap -p <puertos> -sCV <IP> -oN <archivo_salida>
```

| Opciones     | Usos                                    | + Info            |
| ------------ | --------------------------------------- | ----------------- |
| *-p*         | Indicar un puerto objetivo              | `-p 80`           |
| *-p-*        | Escanear todos los puertos              |                   |
| *--open*     | Escanear solo los puertos abiertos      |                   |
| *--min-rate* | Mínimos paquetes a lanzar               | `--min-rate 5000` |
| *-sS*        | Escaneo silencioso                      |                   |
| *-sC*        | Escaneo en busca de scripts explotables |                   |
| *-sV*        | Escaneo en busca de versiones           |                   |
| *-sU*        | Escaneo de UDP                          |                   |
| *-Pn*        | No realizar ping                        |                   |
| *-n*         | No realizar resolución DNS              |                   |
| *-vvv*       | Verbosidad                              |                   |
| *-oN*        | Extraer en formato Nmap                 |                   |
| *-oG*        | Extraer en formato grepeable            |                   |
| *-iL*        | Fichero con direcciones IP              |                   |
| *-A*         | Escaneo Agresivo                        |                   |
| *-T0-4*      | Intensidad del escaneo                  |                   |
