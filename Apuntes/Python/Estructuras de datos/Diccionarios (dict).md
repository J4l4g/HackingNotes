```ad-info
- Colecciones de pares clave:valor.
- Las claves son únicas, los valores pueden repetirse.
- Muy útiles para representar estructuras de datos tipo JSON, tablas, o datos relacionados por clave
  
#### Usos comunes
- Almacenar datos asociados a claves únicas. 
- Buscar valores rápidamente por su clave.  
- Actualizar, agregar o eliminar pares clave-valor.
```


**Cómo se crean:**
```python
miDiccionario = {"a": 1, "b": 2}
otroDiccionario = dict(x=10, y=20)
dicVacio = {}
```


**Cheatsheet básico:**
```python
# Acceder valores
valor = miDiccionario["a"]     # Puede lanzar error si no existe
valorSeguro = miDiccionario.get("a")  # Devuelve None si no existe

# Agregar/Actualizar
miDiccionario["c"] = 3
miDiccionario.update({"d":4, "e":5})

# Eliminar
del miDiccionario["b"]
miDiccionario.pop("c")        # Devuelve el valor eliminado

# Verificación de clave
existe = "a" in miDiccionario  # True o False

# Iterar
for clave, valor in miDiccionario.items():
    print(clave, valor)
``` 
