
`nmap -p- --open -sS --min-rate 5000 -vvv -Pn -n 10.129.2.38 -oG allScan`

```
 [*] IP Address: 10.129.2.38
 [*] Open ports: 22,80,54321
 
```

`nmap -p 22,80,54321 -sCV 10.129.2.38 -oN targeted`


`wfuzz -c --hc 404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://facts.htb/FUZZ`
`http://facts.htb/admin/login`

Creamos una cuenta y accedemos con usuario creado, nos encontramos con un panel de admin de camalen CMS version 2.9.0
Encontramos un campo de subida de ficheros a la hora de editar nuestro perfil

encontramos un exploit en github que nos lo automatiza 
`https://github.com/predyy/CVE-2025-2304/blob/main/exp.py`

Una vez tenemos el acceso como administrador empezamos a hacer reconocimiento de la interfaz de administrador donde encontramos Settings -> General Site -> Filesystem Settings encontrando la clave de acceso y la clave privada de AWS s3, estas dos claves corresponden a usuario y contraseña que identifican a una unidad IAM que pueden correr el servicio s3

estas claves la debemos de añadirlas a MinIO Client con el siguiente comando
`./mcli alias set htb-s3 http://10.129.2.38:54321 AKIA06089A4018ACEB05 mGayyz6RKvweC4T4wbBe970qod2wGfQDqzGUtUYe`

Una vez hemos añadido nuestras keys tenemos que ejecutar un `./mcli ls htb-s3` para listar el contenido de este puerto s3 el contenido se le conoce como buckets, para poder ir listando esto deberemos de ejecutar `./mcli ls htb-s3/<buckets>`



