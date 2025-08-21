#Escalada #PathHijacking
###### Definición
>Forma de elevar privilegios secuestrando comandos

###### Ejemplo
*Por ejemplo, si un binario compilado utiliza el comando* “ls” *sin su ruta absoluta en su código y el atacante crea un archivo malicioso llamado “**ls**” en una de las rutas especificadas en el PATH, el binario ejecutará el archivo malicioso en lugar del comando legítimo* “ls” *cuando sea llamado.*

###### Enumeración
En el binario compilado podemos buscar en el contenido buscando las cadenas de caracteres imprimibles en el binario usando `strings <binary_name>`, después suponiendo como funciona podemos filtar con `strings <binary_name> | grep "<coamdo>"`, si el comando se ejecuta de forma relativa (Sin ruta absoluta) se nos permitirá modificar nuestro PATH para poder escalar privilegios.
Tenemos que enumerar el PATH del usuario actual `echo $PATH` 
###### Explotación
Deberemos manipular el PATH de la siguiente forma `export PATH=`