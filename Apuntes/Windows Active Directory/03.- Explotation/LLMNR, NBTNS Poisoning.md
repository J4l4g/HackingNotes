## Obtención de usuarios con LLMNR/NBT-NS Poisoning
*LLMNR/NBT* hace referencia a (Link-LocalMulticast Name Resolution) que trabaja en el puerto *5355/UDP*.
*NBT-NS* hace referencia a (NetBIOS Name Service) que trabaja en el puerto *137/UDP*.
Estos métodos nombrados son los alternativos a cuando no funciona o falla el DNS.
Nosotros como atacantes podemos interceptar este trafico y así poder obtener un *HASH NetNTLM* el cual con un ataque de fuerza bruta podemos recuperar la contraseña en texto plano llamando a este ataque:

### LLMNR/NBT-NS Poisoning
***DESDE LINUX***
El ataque consiste en los siguientes paso:
1. Un host intenta conectarse al servidor de impresión en `\\print01.inlanefreight.local`, pero accidentalmente escribe en `\\printer01.inlanefreight.local`.
2. El servidor DNS responde, indicando que este host es desconocido.
3. El host luego transmite a toda la red local preguntando si alguien conoce la ubicación de `\\printer01.inlanefreight.local`.
4. El atacante (nosotros con `Responder`en ejecución) responde al host indicando que es el `\\printer01.inlanefreight.local` el que el host está buscando.
5. El host cree esta respuesta y envía una solicitud de autenticación al atacante con un nombre de usuario y un hash de contraseña NTLMv2.
6. Este hash se puede desnudar o usar en un ataque de relé SMB si existen las condiciones adecuadas.

Utilizaremos la herramienta:

#### RESPONDER
[[RESPONDER]] es una herramienta que se encarga de explotar los protocolos *LLMNR*, *NBT-NS* y *MDNS* capturando las credenciales utilizadas para la autenticación.

Una vez lanzada esta herramienta podemos identificar si se encuentra algún hash *NTLM*, en caso de encontrar estos, los guardaremos en un archivo para posteriormente crackearlos.


#### HASHCAT
Una vez tengamos los hashes en un documento se los pasaremos a la herramienta de [[HASHCAT]] este nos los crackeara y nos devolverá las contraseñas de los usuarios.
