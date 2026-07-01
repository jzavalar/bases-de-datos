#### Lectura 07: Joins y Subconsultas

**Dr. Jesús Zavala Ruiz**  
**Última actualización:** 1 de julio de 2026

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

La **unión** de dos conjuntos A y B es el conjunto que contiene todos los elementos que están en A, en B, o en ambos.

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
    ┌─────────┐
    │    A    │
    │  ┌──┐   │
    │  │2 │   │
    │  │3 │   │
    └──┼──┼───┘
       │1 │4  │
       └──┴───┘
         B
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
    ┌─────
    │  A  │     ┌─────┐
    │  ●──┼─────●  B  │
    │     │     │     │
    └─────┘     └─────┘
      (Solo 2 y 3)
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
    ┌─────────┐
    │    A    │
    │  ┌──┐   │
    │  │  │   │
    └──┼──┘   │
       │1     │  ← A - B = {1}
       └──────┘
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

**Cardinalidad:** Si |A| = m y |B| = n, entonces |A × B| = m × n

**Ejemplo numérico:**
```
A = {1, 2, 3}     (|A| = 3)
B = {a, b}        (|B| = 2)

A × B = {(1,a), (1,b), (2,a), (2,b), (3,a), (3,b)}
|A × B| = 3 × 2 = 6 elementos
```

**Representación visual:**
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

---

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

---

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

---

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
Tabla X                    Tabla Y
┌──────────┐             ─────┬─────
│ key │val_x│             │ key │val_y│
├──────────┤             ├─────┼─────┤
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
┌──────────┬──────────┐
│X.key│X.val│Y.key│Y.val│
├──────────┼──────────┤
│  1  │ x1  │  1  │ y1  │  ← key = 1
│  2  │ x2  │  2  │ y2  │  ← key = 2
└─────┴─────┴─────┴─────┘
Total: 2 filas (solo coincidencias)
```

**Paso 3: Proyección final (INNER JOIN)**
```
┌─────┬─────┬─────┐
│ key │val_x│val_y│
├──────────┼─────
│  1  │ x1  │ y1  │
│  2  │ x2  │ y2  │
└─────┴─────┴─────┘
```

---

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

**LEFT JOIN:**
```
Resultado = R ∪ (R ∩ S) = R
```
**Todas** las filas de R, con NULLs donde no hay coincidencia en S.

**RIGHT JOIN:**
```
Resultado = S ∪ (R ∩ S) = S
```
**Todas** las filas de S, con NULLs donde no hay coincidencia en R.

**FULL OUTER JOIN:**
```
Resultado = R ∪ S
```
**Todas** las filas de ambos conjuntos (unión completa).

**Representación visual unificada:**
```
    ┌─────────┐
    │    R    │
    │  ┌───┐  │
    │  │ M │  │  M = R ∩ S (INNER JOIN)
    │  └───  │  R = LEFT JOIN
    └─────────  S = RIGHT JOIN
                 R∪S = FULL OUTER JOIN
```

---

#### 4. Tipos de Joins con Ejemplos Prácticos

##### 4.1. INNER JOIN (Combinación Interna)

El **INNER JOIN** devuelve **únicamente** las filas donde existe coincidencia en **ambas** tablas. Corresponde a la **intersección** de conjuntos.

**Representación visual:**

```
Tabla x                    Tabla y
┌─────┬─────┐             ┌─────┬─────┐
│ key │val_x│             │ key │val_y│
├──────────┤             ├─────┼─────
│  1  │ x1  │──────┐      │  1  │ y1  │
│  2  │ x2  │─────┐│      │  2  │ y2  │
│  3  │ x3  │     ││      │  4  │ y3  │
└─────┴─────┘     ││      └─────┴─────┘
                  ││
                  ↓↓
            INNER JOIN
                  ↓
            ┌──────────┬─────
            │ key │val_x│val_y│
            ├─────┼──────────┤
            │  1  │ x1  │ y1  │
            │  2  │ x2  │ y2  │
            └───────────────
```

**Diagrama de Venn:**
```
    ┌─────┐
    │  x  │     ┌─────
    │  ●───────●  y  │
    │     │     │     │
    └─────     └─────┘
      (Intersección: R ∩ S)
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

---

##### 4.2. LEFT JOIN (Combinación Izquierda)

El **LEFT JOIN** devuelve **todas** las filas de la tabla izquierda (primera), y las filas coincidentes de la tabla derecha. Si no hay coincidencia, se rellena con **NULL**. Corresponde a preservar todo el conjunto izquierdo.

**Representación visual:**

```
Tabla x                    Tabla y
┌─────┬─────┐             ┌──────────
│ key │val_x│             │ key │val_y│
├──────────┤             ├─────┼─────
│  1  │ x1  │──────┐      │  1  │ y1  │
│  2  │ x2  │─────┐│      │  2  │ y2  │
│  3  │ x3  │     ││      │  4  │ y3  │
└──────────┘     ││      └─────┴─────┘
                  ││
                  ↓↓
            LEFT JOIN
                  ↓
            ┌─────┬─────┬─────┐
            │ key │val_x│val_y│
            ├─────┼──────────┤
            │  1  │ x1  │ y1  │
            │  2  │ x2  │ y2  │
            │  3  │ x3  │ NULL│  ← Sin coincidencia
            └─────┴─────┴─────┘
```

**Diagrama de Venn:**
```
    ┌─────────┐
    │    x    │
    │  ┌──┐   │
    │  │●─┼───●  y
    │  └──┘   │
    └─────────
  (Todo x + intersección: R ∪ (R ∩ S))
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

---

##### 4.3. RIGHT JOIN (Combinación Derecha)

El **RIGHT JOIN** es simétrico al LEFT JOIN: devuelve todas las filas de la tabla derecha y las coincidentes de la izquierda.

**Diagrama de Venn:**
```
    ┌─────────┐
    │    y    │
    │   ┌──┐  │
    │   ●─┼──┐│
    │   └──┘  ││
    └─────────│
         x     │
  (Todo y + intersección: S ∪ (R ∩ S))
```

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

---

##### 4.4. FULL OUTER JOIN (Combinación Completa)

El **FULL OUTER JOIN** devuelve **todas** las filas de ambas tablas, con NULLs donde no hay coincidencia. Corresponde a la **unión** de conjuntos.

**Diagrama de Venn:**
```
    ┌─────────┐
    │    x    │
    │  ┌───┐  │
    │  │ ● │  │
    │  └───  │
    ─────────
      (Unión completa: R ∪ S)
```

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

---

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

---

#### 5. Joins Múltiples y Encadenamiento

En la práctica, combinamos **más de dos** tablas simultáneamente. Esto corresponde a operaciones de conjuntos anidadas.

##### 5.1. Ruta Geográfica Completa del Cliente

**Problema:** Obtener el nombre del cliente, su dirección, ciudad y país.

**Análisis de relaciones desde teoría de conjuntos:**
```
customer → address → city → country
   (C)       (A)      (Ci)     (Co)

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
3. `((C ⋈ A)  Ci)  country`: Intersección final con países

Cada JOIN **reduce** el conjunto de datos filtrando por la condición de relación (intersección sucesiva).

---

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

---

#### 6. Subconsultas (Subqueries) desde la Perspectiva de Conjuntos

Una **subconsulta** es una consulta SQL anidada dentro de otra consulta. Se ejecuta primero y su resultado (un conjunto) se usa en la consulta externa.

##### 6.1. Tipos de Subconsultas

##### a) Subconsultas Escalares

Devuelven un **único valor** (una fila, una columna). Corresponden a conjuntos unitarios.

**Ejemplo:** Clientes cuyo gasto total excede el promedio global.

```sql
SELECT 
    customer_id,
    first_name,
    last_name,
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

**Análisis desde teoría de conjuntos:**
1. La subconsulta más interna: Agrupa payments por customer_id y calcula SUM → Conjunto de totales
2. La subconsulta intermedia: Calcula AVG de ese conjunto → Conjunto unitario {promedio}
3. La consulta externa: Filtra customers donde total_spent ∈ {x | x > promedio}

---

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

**¿Cuál es mejor?** Generalmente, el JOIN es más eficiente porque el optimizador de PostgreSQL puede reordenar las operaciones. Sin embargo, las subconsultas son más legibles cuando la lógica es compleja.

**Equivalencia matemática:**
```
IN (subconsulta) ≡ ⋈ (join con filtrado)
```

---

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

---

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

---

#### 7. Joins vs. Subconsultas: Perspectiva de Conjuntos

##### 7.1. Equivalencias Matemáticas

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

##### 7.2. Cuándo Preferir JOINs

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

##### 7.3. Cuándo Preferir Subconsultas

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

##### 7.4. Comparación de Rendimiento desde Conjuntos

**Caso:** Clientes que han rentado más que el promedio.

**Versión con subconsulta correlacionada (LENTA):**
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

**Problema:** La subconsulta se ejecuta N veces (una por cada customer), calculando |{r ∈ Rental | r.customer_id = c.customer_id}| repetidamente.

**Versión con JOIN (RÁPIDA):**
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

**Ventaja:** El JOIN calcula el producto cartesiano una vez y filtra, en lugar de ejecutar N subconsultas.

**Regla general:** En PostgreSQL, el optimizador es muy bueno reescribiendo subconsultas como JOINs. Sin embargo, las subconsultas correlacionadas en WHERE pueden ser problemáticas con grandes volúmenes de datos.

---

#### 8. Técnicas Avanzadas con Perspectiva de Conjuntos

##### 8.1. SELF JOIN (Auto-combinación)

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

---

##### 8.2. UNION, INTERSECT, EXCEPT

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

**Ejemplo visual:**
```
    ┌─────────┐
    │ Rental │
    │  ┌──┐  │
    │  │∩ │  │ Payment
    │  └──┘  │
    └─────────┘

UNION: Todo el área sombreada (R ∪ P)
INTERSECT: Solo el centro (R ∩ P)
EXCEPT: Rental menos intersección (R − P)
```

---

##### 8.3. LATERAL JOIN

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

---

#### 9. Resumen de Conceptos Clave

| Concepto SQL | Operación de Conjuntos | Notación Matemática | Descripción |
|--------------|------------------------|---------------------|-------------|
| **INNER JOIN** | Intersección | R ∩ S | Solo elementos en ambos conjuntos |
| **LEFT JOIN** | Unión con preservación | R ∪ (R ∩ S) | Todo R, NULL donde no hay S |
| **RIGHT JOIN** | Unión con preservación | S ∪ (R ∩ S) | Todo S, NULL donde no hay R |
| **FULL OUTER JOIN** | Unión | R ∪ S | Todos los elementos de ambos |
| **CROSS JOIN** | Producto Cartesiano | R × S | Todas las combinaciones posibles |
| **UNION** | Unión de conjuntos | R ∪ S | Combina sin duplicados |
| **INTERSECT** | Intersección | R ∩ S | Solo elementos comunes |
| **EXCEPT** | Diferencia | R − S | Elementos en R pero no en S |
| **IN (subconsulta)** | Pertenencia | x ∈ S | Verifica si está en el conjunto |
| **EXISTS** | No vaciedad | S ≠ ∅ | Verifica si el conjunto no es vacío |

---

#### 10. Ejercicios Prácticos con Perspectiva de Conjuntos

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

---

#### 11. Recomendaciones de Buenas Prácticas

1. **Piense en términos de conjuntos:** Antes de escribir SQL, identifique qué operación de conjuntos necesita (unión, intersección, diferencia, producto cartesiano)

2. **Siempre use alias descriptivos:** `customer c` en lugar de solo `customer`

3. **Califique todas las columnas:** `c.first_name` en lugar de `first_name`

4. **Use EXISTS en lugar de IN** para subconsultas correlacionadas grandes (más eficiente en verificación de pertenencia)

5. **Evite SELECT * en JOINs:** Especifique solo las columnas necesarias

6. **Índices en claves foráneas:** Asegúrese de que existan índices en las columnas de join (mejora el rendimiento del producto cartesiano filtrado)

7. **Analice el plan de ejecución:** Use `EXPLAIN ANALYZE` para optimizar consultas complejas

8. **Documente la lógica compleja:** Use comentarios CTE o vistas para consultas muy complejas

---

#### 12. Referencias

- PostgreSQL Documentation: https://www.postgresql.org/docs/current/sql-select.html
- Pagila Schema: https://github.com/devrimgunduz/pagila
- Codd, E.F. (1970). *A Relational Model of Data for Large Shared Data Banks*. Communications of the ACM.
- Date, C.J. (2003). *An Introduction to Database Systems*. Addison-Wesley.
- Wickham, Hadley. (2014). *Tidy Data*. Journal of Statistical Software.
- Celko, Joe. (2009). *SQL for Smarties: Advanced SQL Programming*. Morgan Kaufmann.

