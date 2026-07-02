### Lectura 07: Joins y Subconsultas

**Dr. Jesús Zavala Ruiz**  
**Última actualización:** 2 de julio de 2026

---

#### 1. Introducción

En el modelo relacional, la información se distribuye en múltiples tablas para evitar redundancias y garantizar la integridad de los datos. Sin embargo, en la práctica necesitamos combinar información de diferentes tablas para responder preguntas de negocio complejas. Los **joins** (combinaciones) y las **subconsultas** son los mecanismos fundamentales que nos permiten reconstruir esta información fragmentada.

Esta lectura le proporcionará una comprensión profunda de:
- Los fundamentos de teoría de conjuntos que sustentan las operaciones de join
- Los diferentes tipos de joins y su semántica
- Cómo visualizar mentalmente el proceso de combinación
- Cuándo usar joins versus subconsultas
- Técnicas avanzadas de combinación en PostgreSQL

#### 2. Fundamentos de Teoría de Conjuntos

##### 2.1. Conceptos Básicos

Antes de profundizar en los joins, es esencial comprender que **SQL se basa matemáticamente en la teoría de conjuntos**. Una tabla en una base de datos relacional es, esencialmente, un **conjunto de tuplas** (filas).

**Definiciones fundamentales:**

- **Conjunto (Set):** Colección de elementos distintos. En SQL, una tabla es un conjunto de filas.
- **Elemento:** Cada fila individual en una tabla.
- **Atributo:** Cada columna representa una propiedad de los elementos.

**Ejemplo conceptual:**

Sean los conjuntos:
```
A = {1, 2, 3}
B = {2, 3, 4}
```

##### 2.2. Operaciones de Conjuntos

##### a) Unión (∪)

La **unión** de dos conjuntos A y B es el conjunto que contiene todos los elementos que están en A, en B, o en ambos. En lenguaje matemático se expresa de la siguiente forma: x pertenece a A o (OR) x pertenece a B:

```
A ∪ B = {x | x ∈ A ∨ x ∈ B}
```

**Ejemplo:**
```
A = {1, 2, 3}
B = {2, 3, 4}
A ∪ B = {1, 2, 3, 4}
```

**Representación visual (Diagrama de Venn):**
```
       A
    ┌────────┐
    │        │  B
    │    ┌───┼────┐
    │  1 │ 2 │    │
    │    │ 3 │ 4  │
    └────┼───┘    │
         │        │
         └────────┘
 
A ∪ B = {1, 2, 3, 4}
```

**En SQL - Operador UNION:**
```sql
-- Clientes que han rentado OR han hecho pagos
SELECT customer_id FROM rental
UNION
SELECT customer_id FROM payment;
```

##### b) Intersección (∩)

La **intersección** de A y B es el conjunto de elementos que están **en ambos** conjuntos simultáneamente.

```
A ∩ B = {x | x ∈ A ∧ x ∈ B}
```

**Ejemplo:**
```
A = {1, 2, 3}
B = {2, 3, 4}
A ∩ B = {2, 3}
```

**Representación visual:**
```
       A
    ┌────────┐
    │        │  B
    │    ┌───┼────┐
    │  1 │ 2 │    │
    │    │ 3 │ 4  │
    └────┼───┘    │
         │        │
         └────────┘
 
A ∩ B = {2, 3}
```

**En SQL - Operador INTERSECT:**
```sql
-- Clientes que han rentado Y han hecho pagos
SELECT customer_id FROM rental
INTERSECT
SELECT customer_id FROM payment;
```

##### c) Diferencia (− o \)

La **diferencia** A − B es el conjunto de elementos que están en A pero **NO** en B.

```
A − B = {x | x ∈ A ∧ x ∉ B}
```

**Ejemplo:**
```
A = {1, 2, 3}
B = {2, 3, 4}
A − B = {1}
B − A = {4}
```

**Representación visual:**
```
       A
    ┌────────┐
    │        │  B
    │    ┌───┼────┐
    │  1 │ 2 │    │
    │    │ 3 │ 4  │
    └────┼───┘    │
         │        │
         └────────┘
 
A - B = {1}
B − A = {4}
```

**En SQL - Operador EXCEPT:**
```sql
-- Clientes que han rentado pero NO han hecho pagos
SELECT customer_id FROM rental
EXCEPT
SELECT customer_id FROM payment;
```

##### d) Producto Cartesiano (×)

El **producto cartesiano** de A y B es el conjunto de **todos los pares ordenados** posibles donde el primer elemento viene de A y el segundo de B.

```
A × B = {(a, b) | a ∈ A ∧ b ∈ B}
```

**Ejemplo:**
```
A = {1, 2}
B = {a, b}

A × B = {(1,a), (1,b), (2,a), (2,b)}
```

**Representación matricial:**
```
Representación matricial de A × B:

        B
      |  a  |  b  
    --|-----|-----
  A 1 | 1,a | 1,b
    2 | 2,a | 2,b

A × B = {(1,a), (1,b), (2,a), (2,b)}
```

**En SQL - Operador CROSS JOIN:**

```sql
-- Equivalente al producto cartesiano A × B
SELECT a_val, b_val
FROM (VALUES (1), (2)) AS t1(a_val)
CROSS JOIN (VALUES ('a'), ('b')) AS t2(b_val);
```
```text
 a_val | b_val 
-------+-------
     1 | a
     1 | b
     2 | a
     2 | b
```

**Cardinalidad:** Si |A| = m y |B| = n, entonces |A × B| = m × n

**Ejemplo numérico:**
```
A = {1, 2, 3}     (|A| = 3)
B = {a, b}        (|B| = 2)

A × B = {(1,a), (1,b), (2,a), (2,b), (3,a), (3,b)}
|A × B| = 3 × 2 = 6 elementos
```

**Representación matricial:**
```
        a       b
    ┌───────┬───────┐
  1 │ (1,a) │ (1,b) │
    ├───────┼───────┤
  2 │ (2,a) │ (2,b) │
    ├───────┼───────┤
  3 │ (3,a) │ (3,b) │
    └───────┴───────┘
```

**En SQL - CROSS JOIN:**
```sql
-- Todas las combinaciones posibles de películas y tiendas
SELECT f.title, s.store_id
FROM film f
CROSS JOIN store s;
```

Si hay 1000 películas y 2 tiendas: 1000 × 2 = **2000 filas**

##### 2.3. Conjuntos y Tablas en SQL

**Correspondencia directa:**

| Teoría de Conjuntos | SQL / Base de Datos |
|---------------------|---------------------|
| Conjunto | Tabla |
| Elemento | Fila (tupla) |
| Atributo/Propiedad | Columna |
| Unión (∪) | UNION |
| Intersección (∩) | INTERSECT |
| Diferencia (−) | EXCEPT |
| Producto Cartesiano (×) | CROSS JOIN |
| Subconjunto (⊆) | Subconsulta con IN/EXISTS |

**Propiedades importantes:**

1. **Los conjuntos no tienen orden:** Las filas en una tabla no tienen un orden inherente (a menos que use ORDER BY)
2. **Los conjuntos no tienen duplicados:** En SQL puro, una tabla es un conjunto (sin filas duplicadas), aunque en la práctica las tablas son **multiconjuntos** (bags) que permiten duplicados
3. **Cardinalidad:** El número de filas en una tabla


##### 2.4. Relaciones como Subconjuntos del Producto Cartesiano

Una **relación** entre dos conjuntos A y B es un **subconjunto** de su producto cartesiano A × B.

**Ejemplo conceptual:**

Sean:
```
Clientes = {C1, C2, C3}
Productos = {P1, P2}

Producto Cartesiano Completo:
Clientes × Productos = {
    (C1,P1), (C1,P2),
    (C2,P1), (C2,P2),
    (C3,P1), (C3,P2)
}

Relación "Ha Comprado" (subconjunto):
R = {(C1,P1), (C2,P1), (C2,P2)}
```

**En SQL:**
```sql
-- La tabla 'orders' es una relación (subconjunto)
SELECT customer_id, product_id
FROM orders;

-- Resultado:
-- (C1, P1)
-- (C2, P1)
-- (C2, P2)
```

**Aplicación a Joins:**

Un **JOIN** es esencialmente:
1. Calcular el producto cartesiano de dos tablas
2. Filtrar ese producto cartesiano para obtener un subconjunto específico basado en una condición

```
R ⋈ S = σ_condition(R × S)
```

Donde:
- × es el producto cartesiano
- σ (sigma) es la operación de selección/filtrado
- ⋈ (join) es la combinación de ambos

#### 3. Fundamentos Teóricos de los Joins desde la Perspectiva de Conjuntos

##### 3.1. El Join como Operación Relacional

Matemáticamente, un **theta-join** (θ-join) se define como:

```
R ⋈_θ S = σ_θ(R × S)
```

Donde θ es una condición de predicado (generalmente una igualdad).

**Ejemplo paso a paso:**

Tablas:
```
    Tabla X                  Tabla Y
┌─────┬─────┐             ┌─────┬─────┐
│ key │val_x│             │ key │val_y│
├─────┼─────┤             ├─────┼─────┤
│  1  │ x1  │             │  1  │ y1  │
│  2  │ x2  │             │  2  │ y2  │
│  3  │ x3  │             │  4  │ y3  │
└─────┴─────┘             └─────┴─────┘
```

**Paso 1: Producto Cartesiano (X × Y)**
```
┌─────┬─────┬─────┬─────┐
│X.key│X.val│Y.key│Y.val│
├─────┼─────┼─────┼─────┤
│  1  │ x1  │  1  │ y1  │
│  1  │ x1  │  2  │ y2  │
│  1  │ x1  │  4  │ y3  │
│  2  │ x2  │  1  │ y1  │
│  2  │ x2  │  2  │ y2  │
│  2  │ x2  │  4  │ y3  │
│  3  │ x3  │  1  │ y1  │
│  3  │ x3  │  2  │ y2  │
│  3  │ x3  │  4  │ y3  │
└─────┴─────┴─────┴─────┘
Total: 3 × 3 = 9 filas
```

**Paso 2: Aplicar condición (X.key = Y.key)**
```
┌─────┬─────┬─────┬─────┐
│X.key│X.val│Y.key│Y.val│
├─────┼─────┼─────┼─────┤
│  1  │ x1  │  1  │ y1  │  ← key = 1
│  2  │ x2  │  2  │ y2  │  ← key = 2
└─────┴─────┴─────┴─────┘
Total: 2 filas (solo coincidencias)
```

**Paso 3: Proyección final (INNER JOIN)**
```
┌─────┬─────┬─────┐
│ key │val_x│val_y│
├─────┼─────┼─────┤
│  1  │ x1  │ y1  │
│  2  │ x2  │ y2  │
└─────┴─────┴─────┘
```

##### 3.2. Tipos de Join desde la Perspectiva de Conjuntos

Considere los conjuntos:
```
R = {filas de la tabla izquierda}
S = {filas de la tabla derecha}
M = {filas que coinciden en la condición de join}
```

**INNER JOIN:**
```
Resultado = M = R ∩ S
```
Solo las filas que están en **ambos** conjuntos (intersección).

**LEFT JOIN (Left Outer Join):**
```
Resultado = R ⋉ S

Definición formal:
Para cada r ∈ R:
  - Si ∃ s ∈ S tal que r.key = s.key → incluir (r, s)
  - Si no existe tal s → incluir (r, NULL, NULL, ...)

Cardinalidad: |Resultado| ≥ |R|
```
**Todas** las filas de R, extendidas con NULLs donde no hay coincidencia en S.

**RIGHT JOIN (Right Outer Join):**
```
Resultado = R ⋊ S

Definición formal:
Para cada s ∈ S:
  - Si ∃ r ∈ R tal que r.key = s.key → incluir (r, s)
  - Si no existe tal r → incluir (NULL, NULL, ..., s)

Cardinalidad: |Resultado| ≥ |S|
```
**Todas** las filas de S, extendidas con NULLs donde no hay coincidencia en R.

**FULL OUTER JOIN:**
```
Resultado = (R ⋉ S) ∪ (R ⋊ S)

Definición formal:
- Incluir todas las coincidencias (R ∩ S)
- Incluir todas las filas de R sin coincidencia en S (extendidas con NULLs)
- Incluir todas las filas de S sin coincidencia en R (extendidas con NULLs)

Cardinalidad: |Resultado| ≥ max(|R|, |S|)
```
**Todas** las filas de ambos conjuntos, extendidas con NULLs donde no hay coincidencia.

**Representación visual unificada:**
```
       R
    ┌────────┐
    │        │  S
    │    ┌───┼────┐
    │    │ M │    │
    │    │   │    │
    └────┼───┘    │
         │        │
         └────────┘

        M = R ∩ S = INNER JOIN
            R ⋉ S = LEFT JOIN (todo R + coincidencias)
            R ⋊ S = RIGHT JOIN (todo S + coincidencias)
(R ⋉ S) ∪ (R ⋊ S) = FULL OUTER JOIN
```

#### 4. Tipos de Joins con Ejemplos Prácticos

##### 4.1. INNER JOIN (Combinación Interna)

El **INNER JOIN** devuelve **únicamente** las filas donde existe coincidencia en **ambas** tablas. Corresponde a la **intersección** de conjuntos.

**Representación visual:**

```
Tabla X                    Tabla Y
┌─────┬─────┐             ┌─────┬─────┐
│ key │val_x│             │ key │val_y│
├─────┼─────┤             ├─────┼─────┤
│  1  │ x1  │────────────▶│  1  │ y1  │
│  2  │ x2  │────────────▶│  2  │ y2  │
│  3  │ x3  │             │  4  │ y3  │
└─────┴─────┘             └─────┴─────┘
   (solo 1 y 2 tienen coincidencia)

            INNER JOIN
                  ↓
            ┌─────┬─────┬─────┐
            │ key │val_x│val_y│
            ├─────┼─────┼─────┤
            │  1  │ x1  │ y1  │
            │  2  │ x2  │ y2  │
            └─────┴─────┴─────┘
```

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
INNER JOIN y ON x.key = y.key;
```

**Ejemplo con Pagila:**

Obtener información de películas junto con su categoría:

```sql
SELECT 
    f.film_id,
    f.title,
    f.rental_rate,
    c.name AS category
FROM film f
INNER JOIN film_category fc ON f.film_id = fc.film_id
INNER JOIN category c ON fc.category_id = c.category_id
WHERE f.film_id <= 10;
```

**Resultado:**
```
 film_id |        title         | rental_rate |   category    
---------+----------------------+-------------+-------------
       1 | ACADEMY DINOSAUR     |        0.99 | Documentary
       2 | ACE GOLDFINGER       |        4.99 | Action
       3 | ADAPTATION HOLES     |        2.99 | Drama
       4 | AFFAIR PREJUDICE     |        4.99 | Comedy
       5 | AFRICAN EGG          |        2.99 | Comedy
       6 | AGENT TRUMAN         |        2.99 | Documentary
       7 | AIRPLANE SIERRA      |        4.99 | Documentary
       8 | AIRPORT POLLOCK      |        4.99 | Horror
       9 | ALABAMA DEVIL        |        0.99 | Action
      10 | ALADDIN CALENDAR     |        2.99 | Drama
```

**Análisis:** Solo se retornan las películas que **tienen** una categoría asignada. Si una película no tuviera categoría, no aparecería en el resultado. Esto es la **intersección** de los conjuntos "películas" y "categorías".

##### 4.2. LEFT JOIN (Combinación Izquierda)

El **LEFT JOIN** devuelve **todas** las filas de la tabla izquierda (primera), y las filas coincidentes de la tabla derecha. Si no hay coincidencia, se rellena con **NULL**. Corresponde a preservar todo el conjunto izquierdo.

**Representación visual:**

```
Tabla X                    Tabla Y
┌─────┬─────┐             ┌─────┬─────┐
│ key │val_x│             │ key │val_y│
├─────┼─────┤             ├─────┼─────┤
│  1  │ x1  │────────────▶│  1  │ y1  │
│  2  │ x2  │────────────▶│  2  │ y2  │
│  3  │ x3  │             │  4  │ y3  │
└─────┴─────┘             └─────┴─────┘
   (3 no tiene coincidencia)

            LEFT JOIN
                  ↓
            ┌─────┬─────┬─────┐
            │ key │val_x│val_y│
            ├─────┼─────┼─────┤
            │  1  │ x1  │ y1  │
            │  2  │ x2  │ y2  │
            │  3  │ x3  │ NULL│  ← Sin coincidencia
            └─────┴─────┴─────┘
```

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
LEFT JOIN y ON x.key = y.key;
```

**Ejemplo con Pagila - Películas sin inventario:**

Identificar películas que **no** tienen copias físicas en inventario:

```sql
SELECT 
    f.film_id,
    f.title,
    f.rental_rate,
    i.inventory_id
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id
WHERE i.inventory_id IS NULL
LIMIT 10;
```

**Resultado:**
```
 film_id |    title     | rental_rate | inventory_id 
---------+--------------+-------------+--------------
     317 | FELLOWSHIP INN |        4.99 |         NULL
     508 | HUSTLER PARTY  |        4.99 |         NULL
     ...
```

**Análisis desde teoría de conjuntos:**
- El `LEFT JOIN` preserva **todas** las películas (conjunto completo F)
- Cuando una película no tiene inventario, `inventory_id` es `NULL`
- El filtro `WHERE i.inventory_id IS NULL` nos permite identificar la **diferencia de conjuntos**: F − I (películas menos inventario)
- Estos NULLs revelan la **diferencia** entre los conjuntos

**Importancia del NULL:** El valor `NULL` no es un error; es **información valiosa** que indica la ausencia de relación. En auditoría de datos, estos NULLs revelan:
- Problemas de integridad referencial
- Datos incompletos
- Oportunidades de negocio (películas sin stock)
- La **diferencia** entre conjuntos

**Advertencia sobre NULLs:**
Los valores NULL pueden causar comportamientos inesperados:
- `NULL + 5 = NULL` (no 5)
- `NULL = NULL` es `UNKNOWN` (no TRUE ni FALSE)
- `COUNT(columna)` ignora NULLs, pero `COUNT(*)` los cuenta

Use `COALESCE()` para manejar NULLs:
```sql
SELECT COALESCE(i.inventory_id, 0) AS inventory_id
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id;
```

##### 4.3. RIGHT JOIN (Combinación Derecha)

El **RIGHT JOIN** es simétrico al LEFT JOIN: devuelve todas las filas de la tabla derecha y las coincidentes de la izquierda.

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
RIGHT JOIN y ON x.key = y.key;
```

**Nota:** En PostgreSQL, el RIGHT JOIN es menos común porque puede reescribirse como LEFT JOIN intercambiando el orden de las tablas:

```sql
-- RIGHT JOIN
SELECT *
FROM x
RIGHT JOIN y ON x.key = y.key;

-- Equivalente como LEFT JOIN
SELECT *
FROM y
LEFT JOIN x ON x.key = y.key;
```

##### 4.4. FULL OUTER JOIN (Combinación Completa)

El **FULL OUTER JOIN** devuelve **todas** las filas de ambas tablas, con NULLs donde no hay coincidencia. Corresponde a la **unión** de conjuntos.

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
FULL OUTER JOIN y ON x.key = y.key;
```

**Ejemplo con Pagila - Análisis completo de inventario:**

```sql
SELECT 
    f.film_id AS film_id,
    f.title,
    i.inventory_id,
    s.store_id
FROM film f
FULL OUTER JOIN inventory i ON f.film_id = i.film_id
FULL OUTER JOIN store s ON i.store_id = s.store_id
WHERE f.film_id IS NULL OR i.inventory_id IS NULL
LIMIT 10;
```

**Casos de uso:**
- Identificar inconsistencias entre tablas
- Auditoría de integridad referencial
- Reconciliación de datos
- Calcular la **unión** completa de conjuntos

##### 4.5. CROSS JOIN (Producto Cartesiano)

El **CROSS JOIN** genera el producto cartesiano: **todas** las combinaciones posibles. Corresponde exactamente al producto cartesiano × en teoría de conjuntos.

**Ejemplo conceptual:**
```sql
SELECT *
FROM x
CROSS JOIN y;
```

Si `x` tiene 3 filas e `y` tiene 3 filas, el resultado tendrá 3 × 3 = **9 filas**.

**Ejemplo con Pagila - Generar combinaciones película-tienda:**

```sql
SELECT 
    f.film_id,
    f.title,
    s.store_id,
    s.address_id
FROM film f
CROSS JOIN store s
WHERE f.film_id <= 5;
```

**Resultado:**
```
 film_id |        title         | store_id | address_id 
---------+----------------------+----------+------------
       1 | ACADEMY DINOSAUR     |        1 |        1
       1 | ACADEMY DINOSAUR     |        2 |        2
       2 | ACE GOLDFINGER       |        1 |        1
       2 | ACE GOLDFINGER       |        2 |        2
       3 | ADAPTATION HOLES     |        1 |        1
       3 | ADAPTATION HOLES     |        2 |        2
       4 | AFFAIR PREJUDICE     |        1 |        1
       4 | AFFAIR PREJUDICE     |        2 |        2
       5 | AFRICAN EGG          |        1 |        1
       5 | AFRICAN EGG          |        2 |        2
```

**Análisis de cardinalidad:**
- 5 películas × 2 tiendas = **10 combinaciones**
- Esto es exactamente el **producto cartesiano** F × S

**Advertencia:** Use CROSS JOIN con precaución. Con tablas grandes, el resultado puede ser masivo (1000 × 1000 = 1,000,000 filas).

#### 5. Joins Múltiples y Encadenamiento

En la práctica, combinamos **más de dos** tablas simultáneamente. Esto corresponde a operaciones de conjuntos anidadas.

##### 5.1. Ruta Geográfica Completa del Cliente

**Problema:** Obtener el nombre del cliente, su dirección, ciudad y país.

**Análisis de relaciones desde teoría de conjuntos:**
```
customer → address → city → country
   (C)       (A)     (Ci)    (Co)

Resultado = C ⋈ A ⋈ Ci ⋈ Co
```

**Solución:**
```sql
SELECT 
    c.first_name || ' ' || c.last_name AS customer_name,
    a.address,
    a.district,
    ci.city,
    co.country
FROM customer c
INNER JOIN address a ON c.address_id = a.address_id
INNER JOIN city ci ON a.city_id = ci.city_id
INNER JOIN country co ON ci.country_id = co.country_id
WHERE c.customer_id <= 5;
```

**Resultado:**
```
 customer_name |      address       |  district   |    city    |   country   
---------------+--------------------+-------------+------------+-------------
 MARY SMITH    | 1013 Tabuk Boulevard| California | A Coruña   | Spain
 PATRICIA JOHNSON | 932 Santiago de Compostela | Texas | Abha | Saudi Arabia
 LINDA WILLIAMS | 1993 Tabuk Boulevard | California | Adana | Turkey
 BARBARA JONES | 947 Tabuk Boulevard | California | Abu Dhabi | United Arab Emirates
 ELIZABETH BROWN | 1234 Tabuk Boulevard | California | Acapulco | Mexico
```

**Análisis del proceso desde conjuntos:**
1. `customer ⋈ address`: Intersección de clientes con direcciones válidas
2. `(C ⋈ A) ⋈ city`: Intersección del resultado anterior con ciudades
3. `((C ⋈ A) ⋈ Ci) ⋈ Co`: Intersección final con países

Cada JOIN **reduce** el conjunto de datos filtrando por la condición de relación (intersección sucesiva).

##### 5.2. Actores con Más de 40 Películas

**Problema:** Identificar actores que han participado en más de 40 películas.

**Análisis desde teoría de conjuntos:**
```
actor → film_actor → film
 (A)       (FA)      (F)

Paso 1: A ⋈ FA (actores con sus participaciones)
Paso 2: (A ⋈ FA) ⋈ F (agregar información de películas)
Paso 3: GROUP BY + HAVING (filtrar por cardinalidad)
```

**Solución:**
```sql
SELECT 
    a.actor_id,
    a.first_name || ' ' || a.last_name AS actor_name,
    COUNT(fa.film_id) AS film_count
FROM actor a
INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
GROUP BY a.actor_id, a.first_name, a.last_name
HAVING COUNT(fa.film_id) > 40
ORDER BY film_count DESC;
```

**Resultado:**
```
 actor_id |    actor_name    | film_count 
----------+------------------+------------
      107 | JIM MOSTEL       |         45
       23 | WILL WILSON      |         43
       81 | KENNETH PALTROW  |         42
      139 | CUBA ALLEN       |         41
```

**Análisis:** 
- El JOIN conecta actores con sus películas (vía `film_actor`)
- `GROUP BY` agrupa por actor (forma subconjuntos)
- `COUNT` calcula la cardinalidad de cada subconjunto
- `HAVING` filtra los grupos con cardinalidad > 40
- `ORDER BY` ordena descendente

#### 6. Subconsultas (Subqueries) desde la Perspectiva de Conjuntos

Una **subconsulta** es una consulta SQL anidada dentro de otra consulta. Se ejecuta primero y su resultado (un conjunto) se usa en la consulta externa.

##### 6.1. Tipos de Subconsultas

##### a) Subconsultas Escalares

Devuelven un **único valor** (una fila, una columna). Corresponden a conjuntos unitarios.

**Ejemplo:** Clientes cuyo gasto total excede el promedio global.

**Versión con subconsultas escalares (menos eficiente):**
```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    (SELECT SUM(amount) 
     FROM payment p 
     WHERE p.customer_id = c.customer_id) AS total_spent
FROM customer c
WHERE (
    SELECT SUM(amount) 
    FROM payment p 
    WHERE p.customer_id = c.customer_id
) > (
    SELECT AVG(total_amount)
    FROM (
        SELECT customer_id, SUM(amount) AS total_amount
        FROM payment
        GROUP BY customer_id
    ) AS customer_totals
);
```

**Versión optimizada con JOIN (más eficiente):**
```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    customer_totals.total_spent
FROM customer c
INNER JOIN (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM payment
    GROUP BY customer_id
) AS customer_totals ON c.customer_id = customer_totals.customer_id
WHERE customer_totals.total_spent > (
    SELECT AVG(total_amount)
    FROM (
        SELECT customer_id, SUM(amount) AS total_amount
        FROM payment
        GROUP BY customer_id
    ) AS averages
);
```

**Nota:** La primera versión muestra el concepto de subconsultas escalares, pero es ineficiente porque la subconsulta se ejecuta dos veces por cada fila. La segunda versión calcula los totales una sola vez usando un JOIN.

**Análisis desde teoría de conjuntos:**
1. La subconsulta más interna: Agrupa payments por customer_id y calcula SUM → Conjunto de totales
2. La subconsulta intermedia: Calcula AVG de ese conjunto → Conjunto unitario {promedio}
3. La consulta externa: Filtra customers donde total_spent ∈ {x | x > promedio}

##### b) Subconsultas de Columna (con IN)

Devuelven una **columna de valores** para usar con `IN`, que corresponde a la **pertenencia a un conjunto**.

**Ejemplo:** Películas de las categorías 'Action' o 'Comedy':

```sql
SELECT 
    f.film_id,
    f.title,
    f.rental_rate
FROM film f
WHERE f.film_id IN (
    SELECT fc.film_id
    FROM film_category fc
    INNER JOIN category c ON fc.category_id = c.category_id
    WHERE c.name IN ('Action', 'Comedy')
)
LIMIT 10;
```

**Análisis desde teoría de conjuntos:**
- La subconsulta define un conjunto: C = {film_id | category ∈ {'Action', 'Comedy'}}
- La consulta externa filtra: {f ∈ Film | f.film_id ∈ C}
- Esto es una **intersección**: Film ∩ C

**Resultado:**
```
 film_id |        title         | rental_rate 
---------+----------------------+-------------
       2 | ACE GOLDFINGER       |        4.99
       4 | AFFAIR PREJUDICE     |        4.99
       5 | AFRICAN EGG          |        2.99
       9 | ALABAMA DEVIL        |        0.99
      11 | ALASKA PHANTOM       |        4.99
      13 | ALICE FANTASIA       |        4.99
      14 | ALIENS CENTER        |        2.99
      15 | ALLEY EVOLUTION      |        2.99
      16 | ALONE TRIP           |        0.99
      17 | ALONE TRIP           |        0.99
```

**Alternativa con JOIN:**
```sql
SELECT DISTINCT
    f.film_id,
    f.title,
    f.rental_rate
FROM film f
INNER JOIN film_category fc ON f.film_id = fc.film_id
INNER JOIN category c ON fc.category_id = c.category_id
WHERE c.name IN ('Action', 'Comedy')
LIMIT 10;
```

**Nota sobre DISTINCT:** Se usa `DISTINCT` porque una película podría estar en múltiples categorías. Sin `DISTINCT`, si una película estuviera en 'Action' Y 'Comedy', aparecería dos veces en el resultado.

**¿Cuál es mejor?** Generalmente, **el JOIN es más eficiente** porque el optimizador de PostgreSQL puede reordenar las operaciones. Sin embargo, las subconsultas son más legibles cuando la lógica es compleja.

**Equivalencia matemática:**
```
IN (subconsulta) ≡ ⋈ (join con filtrado)
```

##### c) Subconsultas Correlacionadas

Una subconsulta correlacionada **depende** de la consulta externa y se ejecuta **una vez por cada fila** del resultado externo.

**Ejemplo:** Actores que han participado en al menos una película NC-17:

```sql
SELECT 
    a.actor_id,
    a.first_name,
    a.last_name
FROM actor a
WHERE EXISTS (
    SELECT 1
    FROM film_actor fa
    INNER JOIN film f ON fa.film_id = f.film_id
    WHERE fa.actor_id = a.actor_id
    AND f.rating = 'NC-17'
);
```

**Análisis desde teoría de conjuntos:**
- Para cada actor a ∈ Actor, la subconsulta verifica:
  - ∃ f ∈ Film tal que (a, f) ∈ Film_Actor ∧ f.rating = 'NC-17'
- `EXISTS` retorna TRUE si el conjunto resultado no es vacío
- La subconsulta está **correlacionada** porque usa `a.actor_id` de la consulta externa

**Rendimiento:** Las subconsultas correlacionadas pueden ser lentas con grandes volúmenes de datos porque se ejecutan N veces (una por fila). Siempre que sea posible, reescríbalas como JOINs.

##### 6.2. Subconsultas en SELECT

Las subconsultas pueden aparecer en la lista de selección para calcular valores derivados.

**Ejemplo:** Clientes con su total de rentas y el promedio del sistema:

```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    (SELECT COUNT(*) 
     FROM rental r 
     WHERE r.customer_id = c.customer_id) AS rental_count,
    (SELECT AVG(cnt) 
     FROM (
         SELECT customer_id, COUNT(*) AS cnt 
         FROM rental 
         GROUP BY customer_id
     ) AS averages) AS system_average
FROM customer c
WHERE c.customer_id <= 5;
```

**Resultado:**
```
 customer_id | first_name | last_name | rental_count | system_average 
-------------+------------+-----------+--------------+----------------
           1 | MARY       | SMITH     |           10 |        10.5
           2 | PATRICIA   | JOHNSON   |           12 |        10.5
           3 | LINDA      | WILLIAMS  |           11 |        10.5
           4 | BARBARA    | JONES     |            9 |        10.5
           5 | ELIZABETH  | BROWN     |           10 |        10.5
```

**Análisis:**
- La primera subconsulta calcula |{r ∈ Rental | r.customer_id = c.customer_id}| (cardinalidad)  
- La segunda subconsulta calcula el promedio global (conjunto unitario)  

#### 7: Expresiones de Tabla Comunes (CTE)

Una **Expresión de Tabla Común** (Common Table Expression, CTE) es un conjunto de resultados nombrado y temporal que existe únicamente dentro del alcance de una sola instrucción SQL. Los CTEs fueron introducidos en el estándar SQL:1999 y proporcionan una alternativa más legible y mantenible a las subconsultas anidadas.

##### 7.1. Sintaxis Básica

La sintaxis fundamental de un CTE es:

```sql
WITH nombre_cte AS (
    -- Consulta que define el CTE
    SELECT columnas
    FROM tablas
    WHERE condiciones
)
-- Consulta principal que usa el CTE
SELECT *
FROM nombre_cte;
```

**Perspectiva de teoría de conjuntos:**

Un CTE define un conjunto temporal con nombre:
```
CTE = {t | t ∈ Resultado_Consulta}
```

Este conjunto puede ser referenciado múltiples veces en la consulta principal, similar a cómo una variable en programación almacena un valor para reutilización.

##### 7.2. CTEs Simples: Alternativa a Subconsultas

**Problema:** Identificar clientes que han rentado más películas que el promedio.

**Versión con subconsulta (menos legible):**
```sql
SELECT c.customer_id, c.first_name, c.last_name
FROM customer c
WHERE (
    SELECT COUNT(*) 
    FROM rental r 
    WHERE r.customer_id = c.customer_id
) > (
    SELECT AVG(cnt)
    FROM (
        SELECT customer_id, COUNT(*) AS cnt
        FROM rental
        GROUP BY customer_id
    ) AS averages
);
```

**Versión con CTE (más legible):**
```sql
WITH customer_rentals AS (
    -- Paso 1: Contar rentas por cliente
    SELECT 
        customer_id,
        COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
),
average_rentals AS (
    -- Paso 2: Calcular el promedio global
    SELECT AVG(rental_count) AS avg_count
    FROM customer_rentals
)
-- Paso 3: Filtrar clientes above the average
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    cr.rental_count
FROM customer c
INNER JOIN customer_rentals cr ON c.customer_id = cr.customer_id
CROSS JOIN average_rentals ar
WHERE cr.rental_count > ar.avg_count
ORDER BY cr.rental_count DESC;
```

**Resultado:**
```
 customer_id | first_name | last_name | rental_count 
-------------+------------+-----------+--------------
          58 | TAMMY     | SMITH     |           39
         148 | ELEANOR   | HUNT      |           37
         236 | MARCIA    | DEAN      |           36
         526 | KARL      | SEAL      |           35
         178 | MARION    | SNYDER    |           35
...
(150 rows)
```

**Análisis desde teoría de conjuntos:**

```
Paso 1: CR = {(customer_id, COUNT(*)) | customer_id ∈ Rental}
Paso 2: AR = {AVG(cnt) | cnt ∈ CR.rental_count}  (conjunto unitario)
Paso 3: Resultado = {c ∈ Customer | ∃ cr ∈ CR: c.customer_id = cr.customer_id ∧ cr.rental_count > AR}
```

**Ventajas del CTE:**
1. **Legibilidad:** Cada paso está claramente nombrado y documentado
2. **Mantenibilidad:** Más fácil de modificar y depurar
3. **Reutilización:** El CTE `customer_rentals` se usa dos veces sin recalcular
4. **Modularidad:** Cada CTE puede probarse independientemente

##### 7.3. Múltiples CTEs: Encadenamiento de Operaciones

Los CTEs pueden referenciarse entre sí, permitiendo construir consultas complejas paso a paso.

**Problema:** Analizar las ventas por categoría de película, mostrando solo categorías con ventas superiores al promedio.

```sql
WITH category_sales AS (
    -- Paso 1: Calcular ventas totales por categoría
    SELECT 
        c.category_id,
        c.name AS category_name,
        COUNT(r.rental_id) AS total_rentals,
        SUM(p.amount) AS total_revenue
    FROM category c
    INNER JOIN film_category fc ON c.category_id = fc.category_id
    INNER JOIN inventory i ON fc.film_id = i.film_id
    INNER JOIN rental r ON i.inventory_id = r.inventory_id
    INNER JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY c.category_id, c.name
),
average_metrics AS (
    -- Paso 2: Calcular promedios globales
    SELECT 
        AVG(total_rentals) AS avg_rentals,
        AVG(total_revenue) AS avg_revenue
    FROM category_sales
)
-- Paso 3: Filtrar categorías above average
SELECT 
    cs.category_name,
    cs.total_rentals,
    cs.total_revenue,
    ROUND(cs.total_rentals - am.avg_rentals, 2) AS rentals_above_avg,
    ROUND(cs.total_revenue - am.avg_revenue, 2) AS revenue_above_avg
FROM category_sales cs
CROSS JOIN average_metrics am
WHERE cs.total_rentals > am.avg_rentals
   OR cs.total_revenue > am.avg_revenue
ORDER BY cs.total_revenue DESC;
```

**Resultado:**
```
 category_name | total_rentals | total_revenue | rentals_above_avg | revenue_above_avg 
---------------+---------------+---------------+-------------------+-------------------
 Sports        |          1973 |        5312.74 |            589.33 |           1589.41
 Animation     |          1966 |        5304.74 |            582.33 |           1581.41
 Drama         |          1963 |        5299.74 |            579.33 |           1576.41
 Comedy        |          1957 |        5281.74 |            573.33 |           1558.41
 Foreign       |          1941 |        5245.74 |            557.33 |           1522.41
...
(10 rows)
```

**Análisis del flujo de datos:**

```
category_sales:
┌──────────────┬─────────────────┬───────────────┐
│ category_id  │ category_name   │ total_rentals │
├──────────────┼─────────────────┼───────────────┤
│ 1            │ Action          │ 1893          │
│ 2            │ Animation       │ 1966          │
│ 3            │ Children        │ 1842          │
│ ...          │ ...             │ ...           │
└──────────────┴─────────────────┴───────────────┘

average_metrics:
┌──────────────┬───────────────┐
│ avg_rentals  │ avg_revenue   │
├──────────────┼───────────────┤
│ 1383.67      │ 3723.33       │
└──────────────┴───────────────┘

Resultado final:
┌──────────────┬─────────────────┬─────────────────┐
│ category_name│ total_rentals   │ rentals_above   │
├──────────────┼─────────────────┼─────────────────┤
│ Sports       │ 1973            │ 589.33          │
│ Animation    │ 1966            │ 582.33          │
│ ...          │ ...             │ ...             │
└──────────────┴─────────────────┴─────────────────┘
```

##### 7.4. CTEs Recursivos

Los **CTEs recursivos** son una poderosa extensión que permite consultar datos jerárquicos o generar secuencias. Un CTE recursivo se referencia a sí mismo dentro de su definición.

**Sintaxis:**
```sql
WITH RECURSIVE nombre_cte AS (
    -- Caso base (ancla)
    SELECT ...
    UNION ALL
    -- Caso recursivo
    SELECT ...
    FROM nombre_cte
    WHERE condición_de_terminación
)
SELECT * FROM nombre_cte;
```

**Ejemplo 1: Generar una secuencia de números**

```sql
WITH RECURSIVE numbers AS (
    -- Caso base: iniciar con 1
    SELECT 1 AS n
    UNION ALL
    -- Caso recursivo: agregar 1 al número anterior
    SELECT n + 1
    FROM numbers
    WHERE n < 10  -- Condición de terminación
)
SELECT n FROM numbers;
```

**Resultado:**
```
 n  
----
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
(10 rows)
```

**Ejemplo 2: Jerarquía de referencias entre películas**

Aunque Pagila no tiene una jerarquía explícita, podemos simular una cadena de referencias usando películas con títulos relacionados.

**Problema:** Encontrar todas las películas que comienzan con "ACE" y sus "sucesoras" (películas que comienzan con la última letra de la película anterior).

```sql
WITH RECURSIVE film_chain AS (
    -- Caso base: películas que comienzan con 'ACE'
    SELECT 
        film_id,
        title,
        LEFT(title, 3) AS prefix,
        RIGHT(title, 1) AS last_char,
        1 AS level,
        ARRAY[film_id] AS path
    FROM film
    WHERE title LIKE 'ACE%'
    
    UNION ALL
    
    -- Caso recursivo: encontrar películas que comienzan con la última letra
    SELECT 
        f.film_id,
        f.title,
        LEFT(f.title, 3) AS prefix,
        RIGHT(f.title, 1) AS last_char,
        fc.level + 1,
        fc.path || f.film_id
    FROM film f
    INNER JOIN film_chain fc ON LEFT(f.title, 1) = fc.last_char
    WHERE f.film_id != ALL(fc.path)  -- Evitar ciclos
      AND fc.level < 5               -- Limitar profundidad
)
SELECT 
    level,
    film_id,
    title,
    prefix,
    last_char,
    path
FROM film_chain
ORDER BY level, film_id;
```

**Resultado (ejemplo):**
```
 level | film_id |        title         | prefix | last_char |    path     
-------+---------+----------------------+--------+-----------+-------------
     1 |      91 | ACE GOLDFINGER       | ACE    | R         | {91}
     2 |     712 | RIVER OUTLAW         | RIV    | W         | {91,712}
     2 |     721 | ROBBERS JOON         | ROB    | N         | {91,721}
     2 |     734 | ROLLERCOASTER BRING  | ROL    | G         | {91,734}
     3 |     315 | FORWARD TEMPLE       | FOR    | E         | {91,712,315}
     3 |     316 | FRANKENSTEER STRANGER| FRA    | R         | {91,712,316}
...
```

**Análisis del proceso recursivo:**

```
Nivel 1 (Ancla):
  ACE GOLDFINGER → termina en 'R'

Nivel 2 (Recursión 1):
  RIVER OUTLAW → termina en 'W'
  ROBBERS JOON → termina en 'N'
  ROLLERCOASTER BRING → termina en 'G'

Nivel 3 (Recursión 2):
  FORWARD TEMPLE → termina en 'E'
  FRANKENSTEER STRANGER → termina en 'R'
  ...

Cada nivel expande el conjunto de resultados basándose en el nivel anterior.
```

##### 7.5. CTEs vs. Subconsultas: Comparación

| Característica | CTE | Subconsulta |
|----------------|-----|-------------|
| **Legibilidad** | Alta (nombre descriptivo) | Media (anidamiento) |
| **Reutilización** | Sí (múltiples referencias) | No (se repite) |
| **Recursividad** | Sí (WITH RECURSIVE) | No |
| **Rendimiento** | Similar (PostgreSQL optimiza ambos) | Similar |
| **Mantenibilidad** | Alta (modular) | Baja (cambios complejos) |
| **Depuración** | Fácil (probar cada CTE) | Difícil (anidamiento) |

**¿Cuándo usar CTEs?**

✅ **Use CTEs cuando:**
- La consulta tiene múltiples pasos lógicos
- Necesita reutilizar el mismo conjunto de resultados
- Trabaja con datos jerárquicos o recursivos
- La legibilidad es prioritaria
- Necesita documentar el flujo de datos

❌ **Prefiera subconsultas cuando:**
- La lógica es simple y de un solo nivel
- El CTE solo se usa una vez
- Trabaja con bases de datos antiguas (pre-SQL:1999)

### 7.6. CTEs y Materialización

PostgreSQL puede **materializar** o **no materializar** un CTE:

- **Materializado:** El CTE se calcula una vez y se almacena temporalmente
- **No materializado:** El CTE se expande inline (como una macro)

**Control de materialización (PostgreSQL 12+):**

```sql
-- Forzar materialización
WITH customer_data AS MATERIALIZED (
    SELECT customer_id, COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
)
SELECT * FROM customer_data;

-- Forzar inline (no materializar)
WITH customer_data AS NOT MATERIALIZED (
    SELECT customer_id, COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
)
SELECT * FROM customer_data;
```

**Perspectiva de teoría de conjuntos:**

```
Materializado:
  CR = {(customer_id, COUNT(*)) | customer_id ∈ Rental}  (calculado una vez)
  Resultado = {c ∈ Customer | ∃ cr ∈ CR: c.customer_id = cr.customer_id}

No materializado:
  Resultado = {c ∈ Customer | ∃ (SELECT COUNT(*) FROM rental WHERE customer_id = c.customer_id)}
  (el CTE se expande inline, similar a una subconsulta)
```

**Recomendación:** En la mayoría de los casos, deje que PostgreSQL decida automáticamente. Use `MATERIALIZED` solo si el CTE es costoso y se referencia múltiples veces.

##### 7.7. Ejemplo Integrador: Análisis de Ventas Mensuales

**Problema:** Generar un reporte de ventas mensuales con promedios móviles y comparación contra el mes anterior.

```sql
WITH monthly_sales AS (
    -- Paso 1: Calcular ventas mensuales
    SELECT 
        DATE_TRUNC('month', payment_date) AS sale_month,
        COUNT(*) AS total_payments,
        SUM(amount) AS total_revenue,
        AVG(amount) AS avg_payment
    FROM payment
    GROUP BY DATE_TRUNC('month', payment_date)
),
monthly_with_lag AS (
    -- Paso 2: Agregar datos del mes anterior
    SELECT 
        sale_month,
        total_payments,
        total_revenue,
        avg_payment,
        LAG(total_revenue) OVER (ORDER BY sale_month) AS prev_month_revenue,
        LAG(total_payments) OVER (ORDER BY sale_month) AS prev_month_payments
    FROM monthly_sales
)
-- Paso 3: Calcular variaciones
SELECT 
    TO_CHAR(sale_month, 'YYYY-MM') AS month,
    total_payments,
    total_revenue,
    ROUND(avg_payment, 2) AS avg_payment,
    ROUND(total_revenue - prev_month_revenue, 2) AS revenue_change,
    ROUND(
        ((total_revenue - prev_month_revenue) / prev_month_revenue * 100), 
        2
    ) AS revenue_growth_pct,
    total_payments - prev_month_payments AS payments_change
FROM monthly_with_lag
WHERE prev_month_revenue IS NOT NULL
ORDER BY sale_month;
```

**Resultado:**
```
  month   | total_payments | total_revenue | avg_payment | revenue_change | revenue_growth_pct | payments_change 
----------+----------------+---------------+-------------+----------------+--------------------+-----------------
 2005-06  |           1000 |        2850.00 |        2.85 |                |                    |                 
 2005-07  |           5271 |       14235.00 |        2.70 |       11385.00 |              399.47 |            4271
 2005-08  |           4973 |       13430.00 |        2.70 |        -805.00 |               -5.66 |            -298
```

**Análisis del flujo de datos:**

```
monthly_sales:
┌────────────┬────────────────┬───────────────┬─────────────┐
│ sale_month │ total_payments │ total_revenue │ avg_payment │
├────────────┼────────────────┼───────────────┼─────────────┤
│ 2005-05-01 │ 1000           │ 2850.00       │ 2.85        │
│ 2005-06-01 │ 5271           │ 14235.00      │ 2.70        │
│ 2005-07-01 │ 4973           │ 13430.00      │ 2.70        │
└────────────┴────────────────┴───────────────┴─────────────┘

monthly_with_lag:
┌────────────┬────────────────┬───────────────┬─────────────────────┐
│ sale_month │ total_revenue  │ prev_revenue  │ revenue_change      │
├────────────┼────────────────┼───────────────┼─────────────────────┤
│ 2005-05-01 │ 2850.00        │ NULL          │ NULL                │
│ 2005-06-01 │ 14235.00       │ 2850.00       │ 11385.00            │
│ 2005-07-01 │ 13430.00       │ 14235.00      │ -805.00             │
└────────────┴────────────────┴───────────────┴─────────────────────┘
```

##### 7.8. Limitaciones y Consideraciones

1. **Ámbito:** Los CTEs solo existen dentro de la consulta que los define
2. **Rendimiento:** En algunos casos, las subconsultas pueden ser más rápidas
3. **Recursión:** Los CTEs recursivos pueden causar bucles infinitos si no se define correctamente la condición de terminación
4. **Actualizaciones:** Los CTEs son de solo lectura (no se pueden actualizar directamente)

**Precaución con CTEs recursivos:**
```sql
-- PELIGRO: Bucle infinito (sin condición de terminación)
WITH RECURSIVE infinite_loop AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM infinite_loop
    -- Falta: WHERE n < limite
)
SELECT * FROM infinite_loop;
-- Este CTE ejecutará hasta que PostgreSQL limite la recursión
```

##### 7.9. Resumen de CTEs

| Concepto | Descripción | Ejemplo de Uso |
|----------|-------------|----------------|
| **CTE Simple** | Conjunto nombrado temporal | Calcular totales una vez |
| **Múltiples CTEs** | CTEs que se referencian entre sí | Análisis en múltiples pasos |
| **CTE Recursivo** | CTE que se referencia a sí mismo | Jerarquías, secuencias |
| **Materialización** | Control de evaluación | Optimización de rendimiento |

**Regla general:** Use CTEs para mejorar la legibilidad y mantenibilidad de consultas complejas. Para consultas simples de un solo nivel, las subconsultas pueden ser más apropiadas.

#### 8. Joins vs. Subconsultas: Perspectiva de Conjuntos

##### 8.1. Equivalencias Matemáticas

Muchas operaciones pueden expresarse tanto con JOINs como con subconsultas. Son **equivalentes** desde la perspectiva de teoría de conjuntos:

**IN con subconsulta:**
```sql
SELECT * FROM film
WHERE film_id IN (SELECT film_id FROM film_category WHERE category_id = 5);
```

**Equivalente a INTERSECT:**
```
Film ∩ {f | ∃ fc ∈ Film_Category: fc.film_id = f.film_id ∧ fc.category_id = 5}
```

**JOIN equivalente:**
```sql
SELECT DISTINCT f.*
FROM film f
INNER JOIN film_category fc ON f.film_id = fc.film_id
WHERE fc.category_id = 5;
```

##### 8.2. Cuándo Preferir JOINs

✅ **Use JOINs cuando:**  
- Necesite combinar columnas de múltiples tablas  
- La lógica es de "relación" (uno a muchos, muchos a muchos)  
- El rendimiento es crítico (los JOINs suelen ser más eficientes)  
- Necesite preservar filas con LEFT/RIGHT JOIN  
- La operación corresponde a una **intersección** o **producto cartesiano filtrado**  

**Ejemplo:**
```sql
-- Mejor con JOIN (intersección de conjuntos)
SELECT c.first_name, p.amount
FROM customer c
INNER JOIN payment p ON c.customer_id = p.customer_id
WHERE p.amount > 10;
```

##### 8.3. Cuándo Preferir Subconsultas

✅ **Use subconsultas cuando:**  
- Necesite un valor agregado para comparar (promedio, máximo, etc.) → **conjunto unitario**  
- La lógica es de "existencia" (EXISTS, NOT EXISTS) → **verificación de pertenencia**  
- La consulta es más legible como subconsulta  
- Necesite filtrar basado en un conjunto de valores (IN) → **pertenencia a conjunto**  

**Ejemplo:**
```sql
-- Mejor con subconsulta (conjunto de clientes con alto gasto)
SELECT customer_id, first_name
FROM customer
WHERE customer_id IN (
    SELECT customer_id 
    FROM payment 
    GROUP BY customer_id 
    HAVING SUM(amount) > 200
);
```

##### 8.4. Comparación de Rendimiento desde Conjuntos

Identificar los clientes que han realizado más rentas que el promedio del sistema.

Este problema puede resolverse con dos enfoques diferentes que, aunque producen el mismo resultado, tienen un desempeño radicalmente distinto debido a cómo ejecutan las operaciones de conjuntos.

##### 8.4.1. Versión 1: Subconsulta Correlacionada (LENTO)

Esta versión usa una subconsulta en el `WHERE` que se ejecuta **una vez por cada fila** de la tabla `customer`:

```sql
SELECT c.customer_id, c.first_name
FROM customer c
WHERE (
    SELECT COUNT(*) 
    FROM rental r 
    WHERE r.customer_id = c.customer_id
) > (
    SELECT AVG(cnt) 
    FROM (
        SELECT customer_id, COUNT(*) AS cnt 
        FROM rental 
        GROUP BY customer_id
    ) AS averages
);
```

**Análisis del plan de ejecución:**

```sql
EXPLAIN ANALYZE
SELECT c.customer_id, c.first_name
FROM customer c
WHERE (
    SELECT COUNT(*) 
    FROM rental r 
    WHERE r.customer_id = c.customer_id
) > (
    SELECT AVG(cnt) 
    FROM (
        SELECT customer_id, COUNT(*) AS cnt 
        FROM rental 
        GROUP BY customer_id
    ) AS averages
);
```

**Salida:**
```text
Seq Scan on customer c  (cost=0.00..15369.50 rows=200 width=8) (actual time=2.345..45.678 rows=150 loops=1)
  Filter: ((SubPlan 1) > (SubPlan 2))
  Rows Removed by Filter: 449
  SubPlan 1
    ->  Aggregate  (cost=38.25..38.26 rows=1 width=8) (actual time=0.045..0.046 rows=1 loops=599)
          ->  Index Only Scan using idx_fk_customer_id on rental r  (cost=0.28..38.00 rows=100 width=0) (actual time=0.012..0.034 rows=28 loops=599)
                Index Cond: (customer_id = c.customer_id)
                Heap Fetches: 0
  SubPlan 2
    ->  Aggregate  (cost=15293.00..15293.01 rows=1 width=8) (actual time=12.345..12.346 rows=1 loops=1)
          ->  HashAggregate  (cost=15290.00..15291.50 rows=150 width=12) (actual time=10.234..11.567 rows=599 loops=1)
                Group Key: rental.customer_id
                ->  Seq Scan on rental  (cost=0.00..14290.00 rows=200000 width=4) (actual time=0.012..5.678 rows=16044 loops=1)
Planning Time: 0.234 ms
Execution Time: 46.123 ms
```

**Medición con \timing:**
```sql
\timing on

SELECT c.customer_id, c.first_name
FROM customer c
WHERE (
    SELECT COUNT(*) 
    FROM rental r 
    WHERE r.customer_id = c.customer_id
) > (
    SELECT AVG(cnt) 
    FROM (
        SELECT customer_id, COUNT(*) AS cnt 
        FROM rental 
        GROUP BY customer_id
    ) AS averages
);
```

**Salida:**
```
 customer_id | first_name 
-------------+------------
           1 | MARY
           2 | PATRICIA
           3 | LINDA
...
(150 rows)

Time: 46.123 ms
```

**¿Por qué es lenta?**

Observe el valor `loops=599` en `SubPlan 1`. Esto significa que la subconsulta se ejecutó **599 veces** (una por cada cliente en la tabla `customer`). Desde la perspectiva de teoría de conjuntos:

- Para cada cliente c ∈ Customer, calcula |{r ∈ Rental | r.customer_id = c.customer_id}|
- Esto equivale a calcular la cardinalidad de 599 subconjuntos diferentes
- Total de operaciones: 599 escaneos de índice + 599 agregaciones

**Problema fundamental:** La subconsulta correlacionada convierte una operación de conjunto en **N operaciones individuales**, donde N = |Customer|.

##### 8.4.2. Versión 2: JOIN con Tablas Derivadas (RÁPIDO)

Esta versión reescribe la lógica usando `JOIN` con tablas derivadas (subconsultas en el `FROM`):

```sql
SELECT c.customer_id, c.first_name
FROM customer c
INNER JOIN (
    SELECT customer_id, COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
) AS customer_rentals ON c.customer_id = customer_rentals.customer_id
CROSS JOIN (
    SELECT AVG(cnt) AS avg_rentals
    FROM (
        SELECT customer_id, COUNT(*) AS cnt
        FROM rental
        GROUP BY customer_id
    ) AS averages
) AS system_avg
WHERE customer_rentals.rental_count > system_avg.avg_rentals;
```

**Análisis del plan de ejecución:**

```sql
EXPLAIN ANALYZE
SELECT c.customer_id, c.first_name
FROM customer c
INNER JOIN (
    SELECT customer_id, COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
) AS customer_rentals ON c.customer_id = customer_rentals.customer_id
CROSS JOIN (
    SELECT AVG(cnt) AS avg_rentals
    FROM (
        SELECT customer_id, COUNT(*) AS cnt
        FROM rental
        GROUP BY customer_id
    ) AS averages
) AS system_avg
WHERE customer_rentals.rental_count > system_avg.avg_rentals;
```

**Salida:**
```text
Hash Join  (cost=15320.50..15345.75 rows=150 width=8) (actual time=8.234..10.567 rows=150 loops=1)
  Hash Cond: (c.customer_id = customer_rentals.customer_id)
  Join Filter: (customer_rentals.rental_count > system_avg.avg_rentals)
  Rows Removed by Join Filter: 449
  ->  Seq Scan on customer c  (cost=0.00..15.00 rows=599 width=8) (actual time=0.012..0.234 rows=599 loops=1)
  ->  Hash  (cost=15290.00..15290.00 rows=599 width=12) (actual time=7.890..7.890 rows=599 loops=1)
        Buckets: 1024  Batches: 1  Memory Usage: 32kB
        ->  Nested Loop  (cost=0.00..15290.00 rows=599 width=12) (actual time=0.045..7.234 rows=599 loops=1)
              ->  Seq Scan on rental  (cost=0.00..14290.00 rows=200000 width=4) (actual time=0.012..3.456 rows=16044 loops=1)
              ->  Aggregate  (cost=1.00..1.01 rows=1 width=8) (actual time=0.023..0.024 rows=1 loops=1)
Planning Time: 0.456 ms
Execution Time: 10.890 ms
```

**Medición con \timing:**
```sql
\timing on

SELECT c.customer_id, c.first_name
FROM customer c
INNER JOIN (
    SELECT customer_id, COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
) AS customer_rentals ON c.customer_id = customer_rentals.customer_id
CROSS JOIN (
    SELECT AVG(cnt) AS avg_rentals
    FROM (
        SELECT customer_id, COUNT(*) AS cnt
        FROM rental
        GROUP BY customer_id
    ) AS averages
) AS system_avg
WHERE customer_rentals.rental_count > system_avg.avg_rentals;
```

**Salida:**
```
 customer_id | first_name 
-------------+------------
           1 | MARY
           2 | PATRICIA
           3 | LINDA
...
(150 rows)

Time: 10.890 ms
```

**¿Por qué es rápida?**

Observe que todos los valores de `loops=1` en el plan de ejecución. Esto significa que cada operación se ejecuta **una sola vez**:

1. **Seq Scan on rental:** Escanea la tabla rental **1 vez** (16,044 filas)
2. **HashAggregate:** Agrupa por customer_id **1 vez** (produce 599 grupos)
3. **Hash Join:** Combina con customer **1 vez**
4. **CROSS JOIN:** Calcula el promedio **1 vez** (conjunto unitario)

Desde la perspectiva de teoría de conjuntos:
- Calcula el conjunto {customer_id, COUNT(*)} **una vez** para todos los clientes
- Almacena el resultado en una tabla hash temporal
- Realiza una sola operación de join para filtrar

##### 8.4.3. Comparación Directa

| Métrica | Subconsulta Correlacionada | JOIN con Tablas Derivadas | Mejora |
|---------|---------------------------|---------------------------|--------|
| **Planning Time** | 0.234 ms | 0.456 ms | -95% (más lento) |
| **Execution Time** | 46.123 ms | 10.890 ms | **+76% (más rápido)** |
| **Tiempo Total** | 46.357 ms | 11.346 ms | **+75% (más rápido)** |
| **Escaneos de rental** | 599 veces | 1 vez | **99.8% menos** |
| **Operaciones de agregación** | 599 veces | 1 vez | **99.8% menos** |
| **Uso de memoria** | Bajo (fila por fila) | 32kB (hash table) | Más eficiente |

**Nota sobre índices:**
Los tiempos mostrados asumen que existe un índice en `rental.customer_id`:
```sql
CREATE INDEX idx_rental_customer_id ON rental(customer_id);
```
Sin este índice, ambas versiones serían significativamente más lentas.

##### 8.4.4. Análisis desde Teoría de Conjuntos

**Versión lenta (Subconsulta correlacionada):**
```
Resultado = {c ∈ Customer | |{r ∈ Rental | r.customer_id = c.customer_id}| > avg}
```
- Calcula |{r ∈ Rental | r.customer_id = c.customer_id}| para **cada** c ∈ Customer
- Total: |Customer| = 599 cálculos de cardinalidad

**Versión rápida (JOIN):**
```
Paso 1: R = {(customer_id, COUNT(*)) | customer_id ∈ Rental}
Paso 2: A = {AVG(cnt) | cnt ∈ R}  (conjunto unitario)
Paso 3: Resultado = {c ∈ Customer | ∃ r ∈ R: c.customer_id = r.customer_id ∧ r.COUNT > A}
```
- Calcula R **una vez** (599 elementos)
- Calcula A **una vez** (1 elemento)
- Realiza un join **una vez**

##### 8.4.5. Regla General

**En PostgreSQL, el optimizador es muy bueno reescribiendo subconsultas como JOINs. Sin embargo, las subconsultas correlacionadas en WHERE pueden ser problemáticas con grandes volúmenes de datos.**

**Recomendación:** Siempre que sea posible, reescriba subconsultas correlacionadas como JOINs con tablas derivadas. La diferencia de rendimiento aumenta exponencialmente con el tamaño de los datos:

| Tamaño de Customer | Subconsulta Correlacionada | JOIN | Factor de Mejora |
|-------------------|---------------------------|------|------------------|
| 599 filas | 46 ms | 11 ms | 4x |
| 10,000 filas | ~800 ms | ~15 ms | 53x |
| 100,000 filas | ~8,000 ms | ~150 ms | 53x |
| 1,000,000 filas | ~80,000 ms | ~1,500 ms | 53x |

Para bases de datos pequeñas (< 10,000 filas), la diferencia puede ser imperceptible. Para bases de datos grandes (> 100,000 filas), el JOIN puede ser **50-100x más rápido**.

#### 9. Técnicas Avanzadas

##### 9.1. SELF JOIN (Auto-combinación)

Un **SELF JOIN** combina una tabla consigo misma. Corresponde a calcular R × R y filtrar.

**Ejemplo:** Encontrar películas con el mismo año de lanzamiento pero diferente duración.

```sql
SELECT DISTINCT
    f1.title AS film_1,
    f1.release_year,
    f1.length AS length_1,
    f2.title AS film_2,
    f2.length AS length_2
FROM film f1
INNER JOIN film f2 ON f1.release_year = f2.release_year
    AND f1.film_id < f2.film_id  -- Evita duplicados
    AND f1.length != f2.length
WHERE f1.film_id <= 5
LIMIT 5;
```

**Análisis desde teoría de conjuntos:**
- `f1` y `f2` son alias de la misma tabla `film`
- Se calcula Film × Film (producto cartesiano consigo misma)
- La condición `f1.film_id < f2.film_id` evita:
  - Combinar una película consigo misma (diagonal del producto cartesiano)
  - Duplicados simétricos: (f1, f2) y (f2, f1)
- Resultado: Subconjunto de Film × Film donde release_year coincide pero length difiere

##### 9.2. UNION, INTERSECT, EXCEPT

Operaciones de conjunto que combinan resultados de consultas. Corresponden directamente a las operaciones de teoría de conjuntos.

**UNION (∪):** Combina resultados eliminando duplicados.

```sql
-- Clientes que han rentado OR han hecho pagos
-- R ∪ P
SELECT customer_id FROM rental
UNION
SELECT customer_id FROM payment;
```

**INTERSECT (∩):** Solo los valores comunes.

```sql
-- Clientes que han rentado Y han hecho pagos
-- R ∩ P
SELECT customer_id FROM rental
INTERSECT
SELECT customer_id FROM payment;
```

**EXCEPT (−):** Valores de la primera consulta que NO están en la segunda.

```sql
-- Clientes que han rentado pero NO han hecho pagos
-- R − P
SELECT customer_id FROM rental
EXCEPT
SELECT customer_id FROM payment;
```

##### 9.3. LATERAL JOIN

Un **LATERAL JOIN** permite que la subconsulta del lado derecho acceda a columnas del lado izquierdo. Corresponde a una operación de conjuntos dependiente.

**Ejemplo:** Obtener las 3 películas más recientes por actor.

```sql
SELECT 
    a.first_name,
    a.last_name,
    f.title,
    f.release_year
FROM actor a
LEFT JOIN LATERAL (
    SELECT f.title, f.release_year
    FROM film f
    INNER JOIN film_actor fa ON f.film_id = fa.film_id
    WHERE fa.actor_id = a.actor_id
    ORDER BY f.release_year DESC
    LIMIT 3
) f ON true
WHERE a.actor_id <= 3;
```

**Análisis desde teoría de conjuntos:**
- `LATERAL` permite que la subconsulta use `a.actor_id`
- Para cada a ∈ Actor, se calcula:
  - {f ∈ Film | (a, f) ∈ Film_Actor} ordenado por release_year
  - Se toman los primeros 3 elementos (TOP 3)
- Se ejecuta una vez por cada actor
- Resultado: Unión de subconjuntos, uno por actor

#### 10. Resumen de Conceptos Clave

| Concepto SQL | Operación de Conjuntos | Notación Matemática | Descripción |
|--------------|------------------------|---------------------|-------------|
| **INNER JOIN** | Intersección | R ∩ S | Solo elementos en ambos conjuntos |
| **LEFT JOIN** | Left Outer Join | R ⋉ S | Todo R, NULL donde no hay S |
| **RIGHT JOIN** | Right Outer Join | R ⋊ S | Todo S, NULL donde no hay R |
| **FULL OUTER JOIN** | Unión extendida | (R ⋉ S) ∪ (R ⋊ S) | Todos los elementos de ambos |
| **CROSS JOIN** | Producto Cartesiano | R × S | Todas las combinaciones posibles |
| **UNION** | Unión de conjuntos | R ∪ S | Combina sin duplicados |
| **INTERSECT** | Intersección | R ∩ S | Solo elementos comunes |
| **EXCEPT** | Diferencia | R − S | Elementos en R pero no en S |
| **IN (subconsulta)** | Pertenencia | x ∈ S | Verifica si está en el conjunto |
| **EXISTS** | No vaciedad | S ≠ ∅ | Verifica si el conjunto no es vacío |

#### 11. Ejercicios Prácticos con Perspectiva de Conjuntos

##### Ejercicio 1: Ruta Geográfica Completa
Escriba una consulta que muestre:
- Nombre del cliente
- Dirección completa (calle, distrito, ciudad, país)
- Solo clientes activos
- Ordene por país y ciudad

**Pista:** Esto es una intersección sucesiva: Customer ∩ Address ∩ City ∩ Country

##### Ejercicio 2: Actores Prolíficos
Identifique los actores que han participado en más de 40 películas, mostrando:
- Nombre completo del actor
- Número de películas (cardinalidad del conjunto)
- Lista de categorías en las que ha actuado (usar STRING_AGG)

**Pista:** Actor ⋈ Film_Actor ⋈ Film, luego GROUP BY y HAVING COUNT > 40

##### Ejercicio 3: Películas sin Alquiler
Encuentre las películas que **nunca** han sido rentadas, mostrando:
- film_id
- title
- rental_rate
- length

**Pista:** Esto es una diferencia de conjuntos: Film − Rental (use LEFT JOIN + WHERE IS NULL)

##### Ejercicio 4: Clientes por Encima del Promedio
Determine los clientes cuyo monto total de pagos excede el promedio global de pagos, mostrando:
- customer_id
- Nombre completo
- Total pagado
- Promedio del sistema

**Pista:** {c ∈ Customer | total(c) > avg(total)} donde total(c) = Σ{p.amount | p ∈ Payment, p.customer_id = c.customer_id}

##### Ejercicio 5: Inventario por Tienda
Para cada tienda, muestre:
- store_id
- Número total de películas en inventario (cardinalidad)
- Número de películas únicas
- Valor total del inventario (suma de replacement_cost)

**Pista:** Store ⋈ Inventory ⋈ Film, GROUP BY store_id

#### 12. Recomendaciones de Buenas Prácticas

1. **Piense en términos de conjuntos:** Antes de escribir SQL, identifique qué operación de conjuntos necesita (unión, intersección, diferencia, producto cartesiano)

2. **Siempre use alias descriptivos:** `customer c` en lugar de solo `customer`

3. **Califique todas las columnas:** `c.first_name` en lugar de `first_name`

4. **Use EXISTS en lugar de IN** para subconsultas correlacionadas grandes (más eficiente en verificación de pertenencia)

5. **Evite SELECT * en JOINs:** Especifique solo las columnas necesarias

6. **Índices en claves foráneas:** Asegúrese de que existan índices en las columnas de join (mejora el rendimiento del producto cartesiano filtrado)

7. **Analice el plan de ejecución:** Use `EXPLAIN ANALYZE` para optimizar consultas complejas

8. **Documente la lógica compleja:** Use comentarios en CTEs y vistas para consultas muy complejas

#### 13. Referencias

Codd, E. F. (1970). A relational model of data for large shared data banks. *Communications of the ACM*, *13*(6), 377–387. <https://doi.org/10.1145/362384.362685>

Date, C. J. (2003). *An introduction to database systems* (8th ed.). Addison-Wesley.

Gündüz, D. (s.f.). *Pagila: A sample PostgreSQL database* [Source code]. GitHub. <https://github.com/devrimgunduz/pagila>

PostgreSQL Global Development Group. (2026). *SQL SELECT: PostgreSQL 18.3 documentation*. <https://www.postgresql.org/docs/current/sql-select.html>

Wickham, H., Çetinkaya-Rundel, M., & Grolemund, G. (2023). *R for data science*. O'Reilly Media. <https://r4ds.hadley.nz/>
