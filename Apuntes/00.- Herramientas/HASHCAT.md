*Esta herramienta se encarga del crackeo de contraseñas pudiendo obtener un hash en fotmato textPlain (Texto plano).*

###### Crackeo de hashes NTLM a txto plano
```bash
hashcat -m 5600 <fichero_hash> </ruta/wordlist>
```

| Opciones | Usos            | + Info      |
| -------- | --------------- | ----------- |
| *-m*     | Modo de crackeo | `5600=NTLM` |
