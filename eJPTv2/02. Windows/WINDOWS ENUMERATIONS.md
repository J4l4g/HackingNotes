### INFORMACION DEL SISTEMA

| COMANDO      | FUNCION                                                   | +Info       |
| ------------ | --------------------------------------------------------- | ----------- |
| *getuserid*  | ID Usuario                                                | Meterpreter |
| *sysinfo*    | Información del sistema                                   | Meterpreter |
| *show_mount* | Ver discos                                                | Meterpreter |
| *hostname*   | Hostaname del equipo                                      | Shell       |
| *systeminfo* | Toda la información del sistema, incluso acctualizaciones | Shell       |

### USUARIOS Y GRUPOS
| COMANDO                  | FUNCION                               | +Info       |
| ------------------------ | ------------------------------------- | ----------- |
| *getuserid*              | ID Usuario                            | Meterpreter |
| *getprivs*               | Ver privilegios                       | Meterpreter |
| *query user*             | Info usuario actual                   | Shell       |
| *net users*              | Todas las cuentas del sistema         | Shell       |
| *net user  "user"*       | Información sobre el usuario indicado | Shell       |
| *net local group*        | Grupos locales del sistema            | Shell       |
| *net localgroup "group"* | Info grupo                            | Shell       |
Para ver información de mas usuarios podemos poner la sesión en background y usar el modulo `post/windows/gather/enum_logged_on_users`


### INFORMACION DE RED


| COMADO          | FUNCION                                  | +Info |
| --------------- | ---------------------------------------- | ----- |
| *ipconfig*      | Información adaptadores de red           | Shell |
| *ipconfig /all* | Información detallada adaptadores de red | Shell |
| *route print*   | Tabla de enrutamiento                    | Shell |
| *arp -a*        | Dispositivos en la red (pivoting)        | Shell |
| *netstan -ano*  | Servicios en red ejecutandose            | Shell |


### PROCESOS Y SERVICIOS

| COMADO                       | FUNCION                                          | +Info       |
| ---------------------------- | ------------------------------------------------ | ----------- |
| *ps*                         | Procesos en ejecucion                            | Meterpreter |
| *pgrep "name_proces"*        | Obtener identificación del proceso               | Meterpreter |
| *migrate X*                  | Migrar a un proceso                              | Meterpreter |
| *net start*                  | Procesos en segundo plano                        | Shell       |
| *wmic service list brief*    | Servicios en ejecucion                           | Shell       |
| *tasklist /SVC*              | Listar procesos ejecutándose bajo otros procesos | Shell       |
| *schtask /query /fo LIST /v* | Información detallada de cada tarea programada   | Shell       |


### AUTOMATIZACON DE ENUMERACION

Usaremos los siguientes módulos de [[METASPLOIT]], hay que poner la sesión en background primero
-  Mostrar nuestros privilegios: `post/windows/gather/win_privs`
-  Mostrar información usuarios: `post/windows/gather/enum_logged_on_users`
-  Mostar información si es una maquina real o una VM: `post/wiindows/gathere/checkvm`
-  Mostar aplicaciones o programas instalados: `post/windows/gather/enum_applications`
-  Mostar si pertenece a un dominio: `post/windows/gather/enum_computers`
-  Mostar actualizaciones: `psot/windows/gather/enum_patches`
-  Mostar recursos compartidos: `post/windows/gather/enum_shares`
-  



