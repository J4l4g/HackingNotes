
Empezamos la maquina realizando un reconocimiento de puertos y servicios con [[NMAP]]:
`nmap -sS -p- --open --min-rate 5000 -n -Pn -vvv 10.10.11.80 -oG allPorts`

Encontramos los puertos `22, 80, 8080`

Añadimos la maquina al fichero `/etc/hosts`

Navegamos al puerto 