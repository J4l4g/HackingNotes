#API #OWASP #Explotacion 

#### Definición
>Cuando hablamos del abuso de APIs, a lo que nos referimos es a la explotación de vulnerabilidades en las interfaces de programación de aplicaciones (**APIs**) que se utilizan para permitir la comunicación y el intercambio de datos entre diferentes aplicaciones y servicios en una red.

Enumeración de todos los `endpoints` de la API y representarlos en [[POSTMAN]]
En la web vamos recopilando APIs usando `CTRL + Shift + C` para ver las peticiones que se tramitan en la red y copiaremos las URLs de las peticiones en las que ponga `/api/`

En [[POSTMAN]] crearemos una nueva colección con un nombre Identificativo que queramos
	 damos a new -> HTTP -> ponemos el método que corresponda a la petición y copiamos la URL y le damos a SEND (nos dará error 404 seguramente)
En el body tendremos que poner la REQUEST del servidor en formato RAW y en el tipo de texto JSON
Le podemos dar a SEND nuevamente y ya nos dará código de estado 200OK guardamos la petición y le ponemos un nombre descriptivo lo añadimos a la colección en la que vamos a trabajar y guardamos



