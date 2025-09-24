
Empezamos la maquina realizando un reconocimiento de puertos y servicios con [[NMAP]]:
`nmap -sS -p- --open --min-rate 5000 -n -Pn -vvv 10.10.11.80 -oG allPorts`

Encontramos los puertos `22, 80, 8080`

Añadimos la maquina al fichero `/etc/hosts`

Navegamos al puerto `80` por la web y encontramos que en el apartado `Docs` nos redirige a `wiki.editor.htb` para poder verlo correctamente lo volvemos a añadir al `/etc/hosts`

Navegando en la web de `http://wiki.editor.htb/xwiki/bin/view/Main/` vemos que la versión de `xWiki` es la `15.10.8`

Buscamos en internet informacion sobre esta version y encontramso que es vune