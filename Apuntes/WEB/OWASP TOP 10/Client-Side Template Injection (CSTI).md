#CSTI #OWASP #Explotacion  

#### Definición
>**Client-Side Template Injection** (**CSTI**) es una vulnerabilidad de seguridad en la que un atacante puede inyectar código malicioso en una **plantilla de cliente**, que se ejecuta en el **navegador** del usuario en lugar del servidor.
>
  A diferencia del **Server-Side Template Injection** (**SSTI**), en el que la plantilla de servidor se ejecuta en el servidor y es responsable de generar el contenido dinámico, en el CSTI, la plantilla de cliente se ejecuta en el navegador del usuario y se utiliza para generar contenido dinámico en el lado del cliente.
>
  Los atacantes pueden aprovechar una vulnerabilidad de CSTI para inyectar código malicioso en una plantilla de cliente, lo que les permite ejecutar comandos en el navegador del usuario y obtener acceso no autorizado a la aplicación web y a los datos sensibles.

Se pueden usar los payloads de [https://github.com/swisskyrepo/PayloadsAllTheThings]