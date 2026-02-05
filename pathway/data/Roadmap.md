# 🗺️ Roadmap de Traducción Triple  
## SQL ↔️ Pandas ↔️ Polars

---

## 🔹 Fase 1: El ABC de los Datos  
**(Estructura y Selección)**  
Aprenderás a mirar los datos y extraer lo que necesitas.

- **Carga de datos**  
  `read_csv` vs `COPY` vs `scan_csv`

- **Selección básica**  
  `SELECT col1, col2` vs `df[['col1']]` vs `df.select(['col1'])`

- **Alias de columnas**  
  `AS` vs `.rename()`

- **Eliminación de duplicados**  
  `DISTINCT` vs `.drop_duplicates()` vs `.unique()`

- **Limitar resultados**  
  `LIMIT` vs `.head()`

---

## 🔹 Fase 2: El Arte de Filtrar  
**(Lógica y Condicionales)**  
Aprenderás a segmentar la información.

- **Filtros simples**  
  `WHERE` vs `df[df['col'] > x]` vs `.filter()`

- **Operadores lógicos**  
  `AND`, `OR`, `IN`, `BETWEEN` y sus equivalentes

- **Manejo de nulos**  
  `IS NULL` vs `.isna()` vs `.is_null()`

- **Lógica condicional**  
  `CASE WHEN` vs `np.where()` vs `pl.when().then().otherwise()`

---

## 🔹 Fase 3: Agregación y Resumen  
**(Estadística Básica)**  
Aprenderás a convertir filas en métricas.

- **Agregación global**  
  `SUM`, `AVG`, `COUNT`, `MAX`, `MIN`

- **Agrupación**  
  `GROUP BY` vs `.groupby().agg()` vs `.group_by().agg()`

- **Filtros sobre agregados**  
  `HAVING` vs filtrado posterior

---

## 🔹 Fase 4: Relaciones  
**(Joins y Uniones)**  
Aprenderás a conectar diferentes fuentes de información.

- **Tipos de joins**  
  `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`

- **Uniones verticales**  
  `UNION ALL` vs `pd.concat()` vs `pl.concat()`

- **Uniones horizontales**  
  Concatenación por índice o posición

---

## 🔹 Fase 5: Limpieza y Estandarización  
Aprenderás a arreglar datos "sucios".

- **Transformación de texto**  
  `UPPER`, `LOWER`, `TRIM`, `REPLACE`

- **Manejo de fechas**  
  Extraer año, mes, día y diferencias de tiempo

- **Casting de tipos**  
  `CAST(col AS type)` vs `.astype()` vs `.cast()`

---

## 🔹 Fase 6: Análisis Avanzado  
**(Window Functions)**  
Aquí es donde te separas de los principiantes.

- **Particiones globales**  
  `OVER()` vs `.transform('sum')` vs `.over(pl.lit(1))`

- **Particiones específicas**  
  `OVER (PARTITION BY ...)`

- **Orden en ventanas**  
  `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`

- **Desplazamientos**  
  `LAG` y `LEAD` (comparar con la fila anterior/siguiente)

---

## 🔹 Fase 7: Estructura de Código y Optimización  
Aprenderás a escribir código profesional.

- **CTEs (tablas temporales)**  
  `WITH table AS (...)` vs variables intermedias vs Polars Lazy Pipes

- **Subconsultas**  
  Consultas dentro de consultas

- **Lazy Evaluation**  
  El concepto de *plan de ejecución* en SQL y Polars

---

## 🧭 ¿Cómo vamos a trabajar cada módulo?

Para cada punto del roadmap, seguiremos este esquema:

1. **El concepto**  
   Qué vamos a hacer (ej: *"Vamos a filtrar filas"*).

2. **PostgreSQL**  
   La sentencia estándar.

3. **Pandas**  
   La forma clásica (imperativa).

4. **Polars**  
   La forma moderna (declarativa).

5. **Diferencia clave**  
   Un tip rápido de por qué uno es mejor que otro en ciertos casos.

---

¿Te parece bien este orden?  
Si estás de acuerdo, cuando tú digas empezamos con la **Fase 1: Carga y Selección Básica**, y ahí te pediré que creemos las tablas de práctica.
