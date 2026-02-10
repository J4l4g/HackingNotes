#IDORs #OWASP #Explotacion 

#### Definición
>Las **Insecure Direct Object References** (**IDOR**) son un tipo de vulnerabilidad de seguridad que se produce cuando una aplicación web utiliza **identificadores internos** (como números o nombres) para identificar y acceder a recursos (como archivos o datos) y no se valida adecuadamente la autorización del usuario para acceder a ellos.
>
  Por ejemplo, si una aplicación web utiliza un identificador numérico para identificar un registro en una base de datos, un atacante puede intentar adivinar este identificador y acceder a los registros sin la debida autorización. Esto puede permitir a los atacantes acceder a información confidencial, modificar datos, crear cuentas de usuario no autorizadas y realizar otras acciones maliciosas.