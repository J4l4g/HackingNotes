```ad-info
- Recorre iterables de manera ordenada.
- Ideal cuando se conocen los elementos a recorrer.
- Muy útil para listas, tuplas, cadenas o diccionarios.
  
#### Usos comunes
- Iterar sobre colecciones.
- Contar, acumular o aplicar operaciones a todos los elementos.
- Recorrer cadenas de texto o rangos numéricos.
```

**Cómo se usa:**
```python
for variable in iterable:
    # bloque de código
```


**Cheatsheet básico:**
```python
miLista = [1,2,3]
for x in miLista:
    print(x)

for letra in "Python":
    print(letra)

for i in range(5):   # 0 a 4
    print(i)

# break -> salir del bucle
# continue -> saltar a la siguiente iteración
```

