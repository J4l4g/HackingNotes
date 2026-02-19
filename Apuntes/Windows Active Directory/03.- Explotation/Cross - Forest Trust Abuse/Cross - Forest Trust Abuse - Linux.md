#AD #SPN #KERBEROASTING

>Esta vulnerabilidad se puede explotar cuando llegas a un dominio, y no puedes escalar mas para arriba, y hay mas dominios con confianzas

Necesitaremos un usuario con credenciales y el dominio al que queremos llegar, usaremos [[IMPACKET]] con `GetUseSPNs`

```shell
impacket-GetUseSPNs -target-domain <dominio.objetivo> <dominio.actual>/<user>
```

Con el comando anterior enumeraremos los usuarios validos y si utilizamos la opción `-request` al inicio del comando, podremos obtener el hash del ticket `TGS` el cual tenemos que intentar crackear con [[HASHCAT]] con la opción `-m 13100`