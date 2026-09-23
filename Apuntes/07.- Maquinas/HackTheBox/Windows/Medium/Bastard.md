
```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.129.61.47 -oG allPorts
```

Ahora haremos el reconocimiento mas exhaustivo sobre los puertos encontrados `80`, `135` y `49154` usando [[NMAP]]
```shell
nmap -p80,135,49154 -sCV -vvv 10.129.61.47 -oN targeted
```

Observamos que en el puerto `80` encontramos rutas y directorios que existen, también vemos que hay un *Drupal 7*, accederemos a la web a través del navegador.

Antes identificaremos las tecnologías que usa
![[Pasted image 20260923110212.png]]

Ahora ya accederemos a la web a través del navegador
![[Pasted image 20260923110533.png]]

Intentaremos crearnos una cuenta en la sección de `Create new account`
![[Pasted image 20260923110800.png]]

Y al rellenar los campos obtenemos el error que no ha podido ser enviado el mail
![[Pasted image 20260923111059.png]]

