Al iniciar una prueba de penetración lo primero que tendremos que realizar es un reconocimiento pasivo del entorno que vamos a auditar. El objetivo es obtener la mayor información posible del objetivo para a la hora de la explotación poder contar con esta.

La informacion que nos puede inteesar es la siguiente:
- IPspace -> ASN (Autonomous System Number) valido para nuestro dominio.
- Domain Information -> ¿Quién administra el dominio?, ¿Tiene subdominios?, ¿Qué servicios externos tiene?, etc.
- Schema Format -> Informacion que nos pueda permitir obtener una lista de usuarios validos.
- Data Disclousure -> Todos aquellos datos de acceso publico como: `.pdf`, `.ppt`, `.docx`.
- Break Data -> Filtraciones que se hayan realizado del dominio objetivo.

Para la obtencion de toda la anterior informacion utilizaremos las siguientes paginas web:
#### WHO.IS
>https://who.is

#### Hurricane Electric BGPToolkit
>https://bgp.he.net

#### viewDNS.info
>https://viewdns.info


