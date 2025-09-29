
```ad-info
- Almacenar colecciones ordenadas de elementos.
- Permiten duplicados y se pueden modificar (agregar, eliminar, cambiar elementos).
```


**Cómo se crean:**
```python
miLista = [1, 2, 3]        # Lista literal
otraLista = list([4, 5, 6])  # Usando constructor
```


```ad-info
- Mantener una secuencia de elementos que pueda crecer o cambiar.  
- Iterar sobre elementos.   
- Ordenar, buscar o modificar elementos.
```

**Cheatsheet básico:**

```python
# Agregar elementos
miLista.append(4)           # Añade al final
miLista.insert(1, 10)       # Añade en un índice específico
miLista.extend([5,6])       # Añade varios elementos

# Eliminar elementos
miLista.remove(10)           # Elimina por valor (primera coincidencia)
miLista.pop()                # Elimina último elemento
del miLista[0]               # Elimina por índice

# Acceso y búsqueda
elemento = miLista[2]        # Tercer elemento
ultimo = miLista[-1]         # Último elemento
existe = 5 in miLista        # True si está

# Copiar
copia = miLista[:]           # Shallow copy
copia2 = list(miLista)       # Otra forma

```
