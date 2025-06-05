#LDAPi #OWASP #Explotacion 

#### Definición
>Las inyecciones **LDAP** (**Protocolo de Directorio Ligero**) son un tipo de ataque en el que se aprovechan las vulnerabilidades en las aplicaciones web que interactúan con un servidor LDAP. El servidor LDAP es un directorio que se utiliza para almacenar información de usuarios y recursos en una red.
>
  La inyección LDAP funciona mediante la inserción de comandos LDAP maliciosos en los campos de entrada de una aplicación web, que luego son enviados al servidor LDAP para su procesamiento. Si la aplicación web no está diseñada adecuadamente para manejar la entrada del usuario, un atacante puede aprovechar esta debilidad para realizar operaciones no autorizadas en el servidor LDAP.

Para enumerarlo lo podemos hacer los los scripts que trae [[NMAP]]
`nmap -p- --script ldap\* <IP_Objetivo>  `

