
Hacemos un [[00. Apuntes/01. Herramientas/NMAP|NMAP]] hacia la maquina victima
`nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn 172.17.0.2`

Observamos que tienen levantado una pagina web así que accedemos a ella, examinando el código fuente vemos un código brainfuck el cual descifraremos con dcode.fc, y nos da como resultado `bebeaguaqueessano`,
haciendo Fuzzing web con [[00. Apuntes/01. Herramientas/GOBUSTER|GOBUSTER]] obsevamos un directorio `images`
`gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

Accedemos al directorio y observamos una imagen nombrada como `agua_sh.gpg`

En el código fuente de la imagen no hay nada, así que probaremos a conectar por SSH con el usuario agua (como pone en la imagen) y la contraseña `bebeaguaqueessano`, conseguimos acceder pero no somos root.
Veremos el `sudo -l` para ver que comandos podemos ejecutar como root.
![[Pasted image 20241201200107.png]]
Ejecutamos el comando como root
![[Pasted image 20241201200132.png]]

Ahora vamos a cambiar el suid de la bash con el siguiente comado
`! chmod s+u /bin/bash`

Hacemos un exit y podremos ejecutar una bash como root con `bash -p`

