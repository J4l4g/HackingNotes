```ad-info
- Ejecuta un bloque repetidamente mientras se cumpla una condición.
- Ideal cuando no se sabe cuántas veces se repetirá.
  
#### Usos comunes
- Leer entradas hasta que el usuario ingrese un valor válido.
- Ejecutar procesos hasta que se cumpla un criterio de parada.
```

**Cómo se usa:**
```python
while condicion:
    # bloque de código
```


**Cheatsheet básico:**
```python
i = 1
while i <= 5:
    print(i)
    i += 1

# break -> salir del bucle
# continue -> saltar a la siguiente iteración
```