
Empezamos con un escaneo básico de [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.141.33 -oN ScanPorts.txt`

Encontramos levantados los puertos `21 FTP`, `22 SSH` y `80 HTTP`

Vemos que podemos acceder vía `FTP anonymous` una vez hemos accedido `ftp 10.10.141.33` vemos que hay un archivo de texto que se llama `note_to_jake.txt` nos transferimos el archivo a la maquina virtual con él comando `get` y podemos leer que en el archivo se le pide al usuario `jake` que cambie su contraseña. Usamos [[HYDRA]] para hacer fuerza bruta hacia este usuario usando el servicio `ssh`
`hydra -l Jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.141.33`

Nos muestra la contraseña del `ssh` `jake:987654321`

Conectamos vía `ssh jake@10.10.141.33`
Navegando por los directorios encontramos en `/home/holt` la primera flag.
Para ser root usaremos el comando `sudo -l` y nos muestra que tenemos permisos de root sobre `/usr/bin/less` usaremos GTFobins para ver la escalada de privilegios y obtendremos la shell de root.
Encontrado así la flag de root.

