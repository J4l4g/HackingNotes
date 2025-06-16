#ShellShok #OWASP #Explotacion 

#### Definición
>La vulnerabilidad Shellshock se produce en el intérprete de comandos Bash, que es utilizado por muchos sistemas operativos Unix y Linux para ejecutar scripts de shell. El problema radica en la forma en que Bash maneja las variables de entorno. Los atacantes pueden inyectar código malicioso en estas variables de entorno, las cuales Bash ejecuta sin cuestionar su origen o contenido.

Durante la enumeración de la pagina web con herramientas como [[GOBUSTER]] hacemos una enumeración de directorios, si se encuentra u `/cgi-bin/` es una posible señal de ShellShock.

Podemos volver a enumerar la web con 
