
# Reconocimiento

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.9.43 -oG allPorts
```

```shell
[*] IP Address: 10.129.9.43
[*] Open ports: 80,135,445,5985
```

```shell
nmap -p80,135,445,5985 -sCV 10.129.9.43 -oN targeted
```


```shell
nxc smb 10.129.9.43
```

## HTTP
Enumeraremos la web para ver los servicios
```shell
whatweb http://10.129.9.43
```

Accedemos a la IP a traves del navegador y probamos a acceder usando `admin::admim`
![[Pasted image 20260304110713.png]]

Nos encontramos en la web en la zona de Firmware Updates con un mensaje que nos dice que el contenido que subamos va a ser visualizado por otro usuario

Crearemos un fichero `.scf` ya que al subir un archivo y un segundo usuario revisarlo, se puede cargar un fichero malicioso de este tipo, a la hora de que a la hora de llamar 