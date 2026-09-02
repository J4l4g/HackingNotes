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

