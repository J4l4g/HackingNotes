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
Nos encontramos en el usuario *Nadine* un archivo llamado `Confidential.txt` y en el de *Nathan* 