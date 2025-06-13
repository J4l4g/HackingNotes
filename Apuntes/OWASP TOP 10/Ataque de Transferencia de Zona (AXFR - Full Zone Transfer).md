#ZoneTransfer #OWASP #Explotacion 

#### Definición
>Los ataques de transferencia de zona, también conocidos como ataques **AXFR**, son un tipo de ataque que se dirige a los servidores **DNS** (**Domain Name System**) y que permite a los atacantes obtener información sensible sobre los dominios de una organización.
>
  En términos simples, los servidores DNS se encargan de traducir los nombres de dominio legibles por humanos en direcciones IP utilizables por las máquinas. Los ataques AXFR permiten a los atacantes obtener información sobre los registros DNS almacenados en un servidor DNS.
>
  El ataque AXFR se lleva a cabo enviando una solicitud de transferencia de zona desde un servidor DNS falso a un servidor DNS legítimo. Esta solicitud se realiza utilizando el protocolo de transferencia de zona DNS (AXFR), que es utilizado por los servidores DNS para transferir registros DNS de un servidor a otro.

Sirve para saber los subdominios que existen y cuales no y asi poder evitar la fuerza bruta

Primero se pueden enumera los `NS (Name server)` con [[DIG]]
`dig ns @127.0.0.1 jalag.local`

Escanear los servidores de correo
`dig mx @127.0.0.1 jalag.local`

Se utilizara la herramienta de de [[DIG]] indicando el tipo de solicitud que queremos realizar, la IP de la maquina victima y el dominio.
`dig axfr @127.0.0.1 jalag.local`