
```ad-info
- Colecciones desordenadas de elementos únicos.
- No permite duplicados.
- Útiles para operaciones matemáticas: unión, intersección, diferencia.
  
#### Usos comunes
- Eliminar duplicados de una lista.
- Operaciones de pertenencia y conjuntos (matemáticas).
- Filtrar elementos únicos.
```


**Cómo se crean:**
```python
miConjunto = {1, 2, 3}           # Literal
otroConjunto = set([3,4,5])      # Desde lista
conjuntoVacio = set()            # Conjunto vacío
```


**Cheatsheet básico:**
```python
# Agregar elementos
miConjunto.add(4)
miConjunto.update([5,6])  # Añade varios elementos

# Eliminar elementos
miConjunto.remove(3)      # Error si no existe
miConjunto.discard(10)    # No da error si no existe
miConjunto.pop()          # Elimina un elemento aleatorio

# Operaciones de conjuntos
A = {1,2,3}
B = {3,4,5}
union = A | B             # {1,2,3,4,5}
interseccion = A & B      # {3}
diferencia = A - B        # {1,2}
```
