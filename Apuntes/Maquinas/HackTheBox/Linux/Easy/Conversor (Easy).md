
`nmap -p- --open -sS --min-rate 5000 -n -Pn 10.129.15.195 -oG allPorts`

``` shell
[*] IP Address: 10.129.15.195
[*] Open ports: 22,80
```

`nmap -p 22,80 -sCV 10.129.15.195 -oN targeted`

Encontramos una web con un panel de login y registro, hacemos un poco de fuzzing mientras investigamos la web en búsqueda de algo que nos pueda aportar mas información
`wfuzz --hc 404 -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://conversor.htb/FUZZ`

Al no encontrar nada nos creamos una cuneta y accedemos a una web en la que nos convierte archivos XML o XSLT a un formato mas estético

Creamos un archivo de prueba para hacer una enumeracion del servicio y versiones de XSLT
En este primer archivo de prueba enumeramos la version con
```xml
<?xml version="1.0" encoding="UTF-8"?>
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
<body>
<br />Version: <xsl:value-of select="system-property('xsl:version')" />
<br />Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br />Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
</body>
</html>
```

Devolviendonos la version el vendor y la URL de este
```txt
Version: 1.0  
Vendor: libxslt  
Vendor URL: http://xmlsoft.org/XSLT/
```

Buscamos en internet un payload para poder escalar el File Upload a un RCE obteniendo una reverse shell.
EN este casop encontramos el siguiente material en github el cual nos otorga el payload ya creado el cual subiremos estando en escucha desde nuestra maquina y nos devolvera una shell
`https://github.com/Fuzz3d/XSLT-Reverse-Shell-`

En la shell somo s usuario `www-data` el cual tendremos que escalr privilegios

