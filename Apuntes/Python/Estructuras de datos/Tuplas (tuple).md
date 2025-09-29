```ad-info
- Colecciones ordenadas e inmutables (no se pueden modificar después de creadas).  
- Permite duplicados.  
- Útiles para valores que no deben cambiar, como coordenadas, claves de diccionario, o registros.
  
#### Usos comunes
- Retornar múltiples valores desde funciones. 
- Usar como clave en diccionarios (inmutable). 
- Guardar datos constantes.
```

**Cómo se crean:**
```python
miTupla = (1, 2, 3)        # Tupla literal
tuplaVacia = ()             # Tupla vacía
tuplaUnElemento = (5,)      # Tupla de un solo elemento
otraTupla = tuple([4,5,6])  # Desde lista
```


**Cheatsheet básico:**
```python
# Acceso
valor = miTupla[1]           # Segundo elemento
ultimo = miTupla[-1]         # Último elemento

# Operaciones
longitud = len(miTupla)
miTuplaConcatenada = miTupla + (4,5)
repetida = miTupla * 2

# Iteración
for x in miTupla:
    print(x)
```