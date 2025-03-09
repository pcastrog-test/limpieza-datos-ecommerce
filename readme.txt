**Limpieza del Dataset de Carritos Abandonados en un E-commerce de Moda**

---

## 🔍 Paso a Paso para Limpiar los Datos

Cuando trabajamos con datos de e-commerce, es común encontrar errores que pueden afectar nuestros análisis. Vamos a limpiar el dataset de **carritos abandonados** siguiendo un flujo lógico.

### 📌 1. Corregir valores negativos en `ValorCarrito`
**Problema:** Algunos valores de `ValorCarrito` son negativos, lo cual no tiene sentido en una compra real.

**Solución:**
- Revisar si hay valores negativos.
- Convertirlos a positivos si fueron errores de signo.

**Código en Pandas:**
```python
df["ValorCarrito"] = df["ValorCarrito"].abs()
```

---

### 📌 2. Manejar valores faltantes (`Genero`, `Region`, `Dispositivo`)
**Problema:** Faltan datos en algunas columnas clave.

**Solución:**
- Si la columna tiene muchos valores vacíos, podríamos eliminarlos.
- Si solo faltan algunos valores, podemos:
  - Rellenar con la moda (valor más frecuente).
  - Rellenar con "No especificado" o "Desconocido".

**Código en Pandas:**
```python
df["Genero"].fillna("No especificado", inplace=True)
df["Region"].fillna(df["Region"].mode()[0], inplace=True)
df["Dispositivo"].fillna(df["Dispositivo"].mode()[0], inplace=True)
```

---

### 📌 3. Corregir inconsistencias en `ClienteRegistrado` y `ClienteRecurrente`
**Problema:** Hay clientes recurrentes (`Sí`) que aparecen como no registrados.

**Solución:**
- Si un cliente es recurrente (`Sí`), debería estar registrado (`Sí`).
- Corregir la inconsistencia.

**Código en Pandas:**
```python
df.loc[(df["ClienteRegistrado"] == "No") & (df["ClienteRecurrente"] == "Sí"), "ClienteRegistrado"] = "Sí"
```

---

### 📌 4. Eliminar RUTs duplicados
**Problema:** Algunos RUTs aparecen más de una vez, lo que puede significar un error en los datos.

**Solución:**
- Identificar y eliminar duplicados, manteniendo solo la primera aparición.

**Código en Pandas:**
```python
df = df.drop_duplicates(subset=["RUT"], keep="first")
```

---

### 📌 5. Corregir fechas imposibles (`FechaAbandono`)
**Problema:** Hay fechas como "30 de febrero" que no existen.

**Solución:**
- Convertir a formato fecha y eliminar valores erróneos.

**Código en Pandas:**
```python
df["FechaAbandono"] = pd.to_datetime(df["FechaAbandono"], errors="coerce")
df = df.dropna(subset=["FechaAbandono"])
```

---

### 📌 6. Detectar valores atípicos en `ValorCarrito`
**Problema:** Hay pedidos con valores extremadamente altos (ej. $1,000,000+ CLP).

**Solución:**
- Revisar la distribución de valores.
- Filtrar valores fuera de un rango razonable (percentiles 5-95 o 1.5*IQR).

**Código en Pandas:**
```python
q1 = df["ValorCarrito"].quantile(0.05)
q3 = df["ValorCarrito"].quantile(0.95)
df = df[(df["ValorCarrito"] >= q1) & (df["ValorCarrito"] <= q3)]
```

---

### 📌 7. Corregir edades erróneas (`Edad`)
**Problema:** Existen edades de 0 y 99, lo que no es realista.

**Solución:**
- Reemplazar valores fuera del rango 18-75 con la mediana.

**Código en Pandas:**
```python
df.loc[(df["Edad"] < 18) | (df["Edad"] > 75), "Edad"] = df["Edad"].median()
```

---

### 📌 8. Revisar inconsistencias en `CantidadProductos`
**Problema:** En algunas filas, `CantidadProductos` no coincide con el número real de productos en la lista.

**Solución:**
- Contar los productos en la columna `Productos` y corregir el número en `CantidadProductos`.

**Código en Pandas:**
```python
df["CantidadProductos"] = df["Productos"].apply(lambda x: len(str(x).split("|")))
```

---

### 📌 9. Corregir categorías mal asignadas
**Problema:** Algunos productos están en categorías incorrectas (ej. "Vestido largo rojo" en "Ropa hombre").

**Solución:**
- Mapear productos a categorías correctas con una tabla de referencia.

**Código en Pandas:**
```python
correcciones_categorias = {
    "Vestido largo rojo": "Ropa mujer",
    "Zapatos de vestir marrón": "Calzado",
    "Chaqueta de cuero negra": "Ropa hombre"
}

df["CategoriasPrincipales"] = df["Productos"].apply(lambda x: correcciones_categorias.get(x.split("|")[0], df["CategoriasPrincipales"]))
```

---

## 🎯 Resumen: Pasos Claves en la Limpieza
1️⃣ Corregir valores negativos en `ValorCarrito`
2️⃣ Manejar valores faltantes con la moda o valores por defecto
3️⃣ Resolver inconsistencias entre `ClienteRegistrado` y `ClienteRecurrente`
4️⃣ Eliminar RUTs duplicados
5️⃣ Arreglar fechas imposibles
6️⃣ Filtrar valores atípicos en `ValorCarrito`
7️⃣ Corregir edades fuera del rango válido
8️⃣ Ajustar `CantidadProductos` con los productos listados
9️⃣ Corregir categorías erróneas

Este proceso **simula una limpieza de datos real** en Growth Hacking, donde los datos de carritos abandonados deben estar en buenas condiciones para **segmentar clientes y optimizar campañas de remarketing**.

