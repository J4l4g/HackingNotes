| COMANDO         | FUNCION                        | +Info |
| --------------- | ------------------------------ | ----- |
| */bin/bash -i*  | Obtener sesión interactiva     | Shell |
| *sessions -u X* | Upgradear sesión a Meterpreter | Shell |
### INFORMACION DEL SISTEMA

| COMANDO                 | FUNCION                                | +Info       |
| ----------------------- | -------------------------------------- | ----------- |
| *sysinfo*               | Información del sistema                | Meterpreter |
| *hostname*              | Hostanme del sistema                   | Shell       |
| **cat /etc/*release***  | Obtener info de la distribucion        | Shell       |
| *uname -a*              | Hostname info del kernel               | Shell       |
| *uname -r*              | Versión del kernel                     | Shell       |
| *env*                   | Variables de entorno                   | Shell       |
| *lscpu*                 | Arquitectura CPU                       | Shell       |
| *df -h*                 | Sistema de archivos y almacenamiento   | Shell       |
| *df -ht "nombre_disco"* | Información de los discos              | Shell       |
| *lsblj \| grep sd*      | Información del almacenamiento         | Shell       |
| *dpkg -l*               | Informacion de los paquetes instalados | Shell       |
| *ls -la /etc/cron.d*    | Tareas programadas                     |             |

### USUARIOS Y GRUPOS


| COMANDO           | FUNCION                       | +Info       |
| ----------------- | ----------------------------- | ----------- |
| *getuserid*       | ID Usuario                    | Meterpreter |
| *whoami*          | Saber que usuario soy         |             |
| *groups "user"*   | Mostar grupo al que pertenece |             |
| *cat /etc/group*  | Mostar info grupos            |             |
| *groups*          | Mostar grupos                 |             |
| *cat /etc/passwd* | Mostrar info de usuarios      |             |
| *last*            | Inicios de sesion legitima    |             |
| *lastlog*         | Ultimos inicios de sesion     |             |


### INFORMACION DE RED


| COMANDO                | FUNCION                     | +Info       |
| ---------------------- | --------------------------- | ----------- |
| *ifconfig*             | Listar interfaces           | Meterpreter |
| *netstat*              | Lista de conexiones         | Meterpreter |
| *route*                | Tabla enrutamiento          | Meterpreter |
| *ip a*                 | Listar interfaces           | Shell       |
| *cat /etc/networks*    | Info de redes               | Shell       |
| *cat /etc/hosts*       | Info de los hosts de red    | Shell       |
| *cat /etc/resolv.conf* | Info DNS                    | Shell       |
| *arp*                  | Identificar sistemas en red | Meterpreter |
