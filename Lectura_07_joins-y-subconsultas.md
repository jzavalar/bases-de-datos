### Lectura 07: Joins y Subconsultas

**Dr. Jesús Zavala Ruiz**  
**Última actualización:** 2 de julio de 2026

---

#### 1. Introducción

En el modelo relacional, la información se distribuye en múltiples tablas para evitar redundancias y garantizar la integridad de los datos. Sin embargo, en la práctica necesitamos combinar información de diferentes tablas para responder preguntas de negocio complejas. Los **joins** (combinaciones) y las **subconsultas** son los mecanismos fundamentales que nos permiten reconstruir esta información fragmentada.

Esta lectura le proporcionará una comprensión profunda de:
- Los fundamentos de teoría de conjuntos que sustentan las operaciones de join
- Los diferentes tipos de joins y su semántica
- Cuándo usar joins, subconsultas o CTEs
- Técnicas avanzadas de combinación en PostgreSQL

#### 2. Fundamentos de Teoría de Conjuntos

##### 2.1. Conceptos Básicos

**SQL se basa matemáticamente en la teoría de conjuntos**. Una tabla en una base de datos relacional es, esencialmente, un **conjunto de tuplas** (filas).

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

Las cuatro operaciones fundamentales de teoría de conjuntos se mapean directamente a operaciones SQL:

| Operación | Notación | Definición | SQL |
|-----------|----------|------------|-----|
| **Unión** | A ∪ B | {x \| x ∈ A ∨ x ∈ B} | UNION |
| **Intersección** | A ∩ B | {x \| x ∈ A ∧ x ∈ B} | INTERSECT |
| **Diferencia** | A − B | {x \| x ∈ A ∧ x ∉ B} | EXCEPT |
| **Producto Cartesiano** | A × B | {(a,b) \| a ∈ A ∧ b ∈ B} | CROSS JOIN |

**Representación visual unificada (Diagrama de Venn):**
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

A ∪ B = {1, 2, 3, 4}     (Unión: todo)
A ∩ B = {2, 3}           (Intersección: común)
A − B = {1}              (Diferencia: solo A)
B − A = {4}              (Diferencia: solo B)
```

**Ejemplos SQL con Pagila:**

```sql
-- UNIÓN: Clientes que han rentado O han hecho pagos
SELECT customer_id FROM rental
UNION
SELECT customer_id FROM payment;

-- INTERSECCIÓN: Clientes que han rentado Y han hecho pagos
SELECT customer_id FROM rental
INTERSECT
SELECT customer_id FROM payment;

-- DIFERENCIA: Clientes que han rentado pero NO han hecho pagos
SELECT customer_id FROM rental
EXCEPT
SELECT customer_id FROM payment;
```

##### 2.3. Producto Cartesiano

El **producto cartesiano** de A y B es el conjunto de **todos los pares ordenados** posibles:

```
A × B = {(a, b) | a ∈ A ∧ b ∈ B}
```

**Representación matricial:**
```
A = {1, 2, 3}     (|A| = 3)
B = {a, b}        (|B| = 2)

        a       b
    ┌───────┬───────┐
  1 │ (1,a) │ (1,b) │
    ├───────┼───────┤
  2 │ (2,a) │ (2,b) │
    ├───────┼───────┤
  3 │ (3,a) │ (3,b) │
    └───────┴───────┘

|A × B| = 3 × 2 = 6 elementos
```

**En SQL - CROSS JOIN:**
```sql
-- Todas las combinaciones posibles de películas y tiendas
SELECT f.title, s.store_id
FROM film f
CROSS JOIN store s;
```

Si hay 1000 películas y 2 tiendas: 1000 × 2 = **2000 filas**

**Advertencia:** Use CROSS JOIN con precaución. Con tablas grandes, el resultado puede ser masivo.

##### 2.4. Relaciones como Subconjuntos del Producto Cartesiano

Una **relación** entre dos conjuntos A y B es un **subconjunto** de su producto cartesiano A × B.

**Aplicación a Joins:**

Un **JOIN** es esencialmente:
1. Calcular el producto cartesiano de dos tablas
2. Filtrar ese producto cartesiano para obtener un subconjunto específico

```
R ⋈ S = σ_condition(R × S)
```

Donde:
- × es el producto cartesiano
- σ (sigma) es la operación de selección/filtrado
- ⋈ (join) es la combinación de ambos

---

#### 3. Tipos de Joins

##### 3.1. Definiciones Formales

Considere los conjuntos:
```
R = {filas de la tabla izquierda}
S = {filas de la tabla derecha}
M = {filas que coinciden en la condición de join}
```

| Tipo de Join | Notación | Definición | Cardinalidad |
|--------------|----------|------------|--------------|
| **INNER JOIN** | R ∩ S | Solo coincidencias | ≤ min(\|R\|, \|S\|) |
| **LEFT JOIN** | R ⋉ S | Todo R + coincidencias | ≥ \|R\| |
| **RIGHT JOIN** | R ⋊ S | Todo S + coincidencias | ≥ \|S\| |
| **FULL OUTER JOIN** | (R ⋉ S) ∪ (R ⋊ S) | Todo R ∪ todo S | ≥ max(\|R\|, \|S\|) |

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

##### 3.2. INNER JOIN

El **INNER JOIN** devuelve **únicamente** las filas donde existe coincidencia en **ambas** tablas.

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
INNER JOIN y ON x.key = y.key;
```

**Ejemplo con Pagila:** Películas con su categoría

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

##### 3.3. LEFT JOIN

El **LEFT JOIN** devuelve **todas** las filas de la tabla izquierda, y las filas coincidentes de la tabla derecha. Si no hay coincidencia, se rellena con **NULL**.

**Sintaxis SQL:**
```sql
SELECT x.key, x.val_x, y.val_y
FROM x
LEFT JOIN y ON x.key = y.key;
```

**Ejemplo con Pagila:** Películas sin inventario

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
 film_id |    title       | rental_rate | inventory_id 
---------+----------------+-------------+--------------
     317 | FELLOWSHIP INN |        4.99 |         NULL
     508 | HUSTLER PARTY  |        4.99 |         NULL
     ...
```

**Análisis desde teoría de conjuntos:**
- El `LEFT JOIN` preserva **todas** las películas (conjunto completo F)
- El filtro `WHERE i.inventory_id IS NULL` identifica la **diferencia de conjuntos**: F − I
- Los NULLs revelan la **ausencia** de relación

**Importancia del NULL:** El valor `NULL` no es un error; es **información valiosa** que indica la ausencia de relación. En auditoría de datos, estos NULLs revelan:
- Problemas de integridad referencial
- Datos incompletos
- Oportunidades de negocio (películas sin stock)

**Advertencia sobre NULLs:**
- `NULL + 5 = NULL` (no 5)
- `NULL = NULL` es `UNKNOWN` (no TRUE ni FALSE)
- `COUNT(columna)` ignora NULLs, pero `COUNT(*)` los cuenta

Use `COALESCE()` para manejar NULLs:
```sql
SELECT COALESCE(i.inventory_id, 0) AS inventory_id
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id;
```

##### 3.4. RIGHT JOIN y FULL OUTER JOIN

El **RIGHT JOIN** es simétrico al LEFT JOIN. En PostgreSQL, es menos común porque puede reescribirse como LEFT JOIN intercambiando el orden de las tablas.

El **FULL OUTER JOIN** devuelve **todas** las filas de ambas tablas, con NULLs donde no hay coincidencia.

**Ejemplo con Pagila:** Análisis completo de inventario

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

#### 4. Joins Múltiples y Encadenamiento

En la práctica, combinamos **más de dos** tablas simultáneamente.

##### 4.1. Ruta Geográfica Completa del Cliente

**Problema:** Obtener el nombre del cliente, su dirección, ciudad y país.

**Análisis desde teoría de conjuntos:**
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
 MARY SMITH    | 1013 Tabuk Blvd    | California  | A Coruña   | Spain
 PATRICIA J.   | 932 Santiago St    | Texas       | Abha       | Saudi Arabia
 LINDA W.      | 1993 Tabuk Blvd    | California  | Adana      | Turkey
 BARBARA J.    | 947 Tabuk Blvd     | California  | Abu Dhabi  | UAE
 ELIZABETH B.  | 1234 Tabuk Blvd    | California  | Acapulco   | Mexico
```

Cada JOIN **reduce** el conjunto de datos filtrando por la condición de relación (intersección sucesiva).

##### 4.2. Actores con Más de 40 Películas

**Problema:** Identificar actores que han participado en más de 40 películas.

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
- El JOIN conecta actores con sus películas
- `GROUP BY` agrupa por actor (forma subconjuntos)
- `COUNT` calcula la cardinalidad de cada subconjunto
- `HAVING` filtra los grupos con cardinalidad > 40

#### 5. Subconsultas

Una **subconsulta** es una consulta SQL anidada dentro de otra consulta. Se ejecuta primero y su resultado (un conjunto) se usa en la consulta externa.

##### 5.1. Tipos de Subconsultas

##### 5.1.1. Subconsultas Escalares

Devuelven un **único valor** (conjunto unitario).

**Ejemplo:** Clientes cuyo gasto total excede el promedio global.

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

##### 5.1.2. Subconsultas de Columna (con IN)

Devuelven una **columna de valores** para usar con `IN` (pertenencia a conjunto).

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
- La subconsulta define: C = {film_id | category ∈ {'Action', 'Comedy'}}
- La consulta externa filtra: {f ∈ Film | f.film_id ∈ C}
- Esto es una **intersección**: Film ∩ C

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

**Nota:** Se usa `DISTINCT` porque una película podría estar en múltiples categorías.

**Equivalencia matemática:**
```
IN (subconsulta) ≡ ⋈ (join con filtrado)
```

##### 5.1.3. Subconsultas Correlacionadas

Una subconsulta correlacionada **depende** de la consulta externa y se ejecuta **una vez por cada fila**.

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
- Para cada actor a ∈ Actor, verifica: ∃ f ∈ Film tal que (a, f) ∈ Film_Actor ∧ f.rating = 'NC-17'
- `EXISTS` retorna TRUE si el conjunto resultado no es vacío

**Rendimiento:** Las subconsultas correlacionadas pueden ser lentas. Siempre que sea posible, reescríbalas como JOINs.

##### 5.2. Subconsultas en SELECT

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

#### 6. Expresiones de Tabla Comunes (CTE)

Una **Expresión de Tabla Común** (*Common Table Expression*, *CTE*) es un conjunto de resultados nombrado y temporal que existe únicamente dentro del alcance de una sola instrucción SQL. Los CTEs proporcionan una alternativa más legible y mantenible a las subconsultas anidadas.

##### 6.1. Sintaxis Básica

```sql
WITH nombre_cte AS (
    SELECT columnas
    FROM tablas
    WHERE condiciones
)
SELECT *
FROM nombre_cte;
```

**Perspectiva de teoría de conjuntos:**
```
CTE = {t | t ∈ Resultado_Consulta}
```

##### 6.2. CTEs Simples: Alternativa a Subconsultas

**Problema:** Identificar clientes que han rentado más películas que el promedio.

**Versión con CTE (más legible):**
```sql
WITH customer_rentals AS (
    SELECT 
        customer_id,
        COUNT(*) AS rental_count
    FROM rental
    GROUP BY customer_id
),
average_rentals AS (
    SELECT AVG(rental_count) AS avg_count
    FROM customer_rentals
)
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
          58 | TAMMY      | SMITH     |           39
         148 | ELEANOR    | HUNT      |           37
         236 | MARCIA     | DEAN      |           36
         526 | KARL       | SEAL      |           35
         178 | MARION     | SNYDER    |           35
...
(150 rows)
```

**Ventajas del CTE:**
1. **Legibilidad:** Cada paso está claramente nombrado
2. **Mantenibilidad:** Más fácil de modificar y depurar
3. **Reutilización:** El CTE se usa múltiples veces sin recalcular
4. **Modularidad:** Cada CTE puede probarse independientemente

##### 6.3. Múltiples CTEs: Encadenamiento

Los CTEs pueden referenciarse entre sí, permitiendo construir consultas complejas paso a paso.

**Problema:** Analizar ventas por categoría, mostrando solo categorías superiores al promedio.

```sql
WITH category_sales AS (
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
    SELECT 
        AVG(total_rentals) AS avg_rentals,
        AVG(total_revenue) AS avg_revenue
    FROM category_sales
)
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

##### 6.4. CTEs Recursivos

Los **CTEs recursivos** permiten consultar datos jerárquicos o generar secuencias.

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
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM numbers
    WHERE n < 10
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

**Ejemplo 2: Jerarquía de películas**

**Problema:** Encontrar películas que comienzan con "ACE" y sus "sucesoras" (películas que comienzan con la última letra de la película anterior).

```sql
WITH RECURSIVE film_chain AS (
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
    
    SELECT 
        f.film_id,
        f.title,
        LEFT(f.title, 3) AS prefix,
        RIGHT(f.title, 1) AS last_char,
        fc.level + 1,
        fc.path || f.film_id
    FROM film f
    INNER JOIN film_chain fc ON LEFT(f.title, 1) = fc.last_char
    WHERE f.film_id != ALL(fc.path)
      AND fc.level < 5
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
     3 |     315 | FORWARD TEMPLE       | FOR    | E         | {91,712,315}
...
```

**Precaución:** Los CTEs recursivos pueden causar bucles infinitos si no se define correctamente la condición de terminación.

##### 6.5. CTEs vs. Subconsultas: Comparación

| Característica | CTE | Subconsulta |
|----------------|-----|-------------|
| **Legibilidad** | Alta (nombre descriptivo) | Media (anidamiento) |
| **Reutilización** | Sí (múltiples referencias) | No (se repite) |
| **Recursividad** | Sí (WITH RECURSIVE) | No |
| **Rendimiento** | Similar | Similar |
| **Mantenibilidad** | Alta (modular) | Baja (cambios complejos) |

**¿Cuándo usar CTEs?**

✅ **Use CTEs cuando:**
- La consulta tiene múltiples pasos lógicos
- Necesita reutilizar el mismo conjunto de resultados
- Trabaja con datos jerárquicos o recursivos
- La legibilidad es prioritaria

❌ **Prefiera subconsultas cuando:**
- La lógica es simple y de un solo nivel
- El CTE solo se usa una vez

##### 6.6. Materialización de CTEs

PostgreSQL puede **materializar** o **no materializar** un CTE (PostgreSQL 12+):

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

**Recomendación:** En la mayoría de los casos, deje que PostgreSQL decida automáticamente.

#### 7. Joins vs. Subconsultas: Perspectiva de Conjuntos

##### 7.1. Equivalencias Matemáticas

Muchas operaciones pueden expresarse tanto con JOINs como con subconsultas:

**IN con subconsulta:**
```sql
SELECT * FROM film
WHERE film_id IN (SELECT film_id FROM film_category WHERE category_id = 5);
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
- El rendimiento es crítico  
- Necesite preservar filas con LEFT/RIGHT JOIN  

**Ejemplo:**
```sql
SELECT c.first_name, p.amount
FROM customer c
INNER JOIN payment p ON c.customer_id = p.customer_id
WHERE p.amount > 10;
```

##### 7.3. Cuándo Preferir Subconsultas

✅ **Use subconsultas cuando:**  
- Necesite un valor agregado para comparar (promedio, máximo, etc.)  
- La lógica es de "existencia" (EXISTS, NOT EXISTS)  
- La consulta es más legible como subconsulta  
- Necesite filtrar basado en un conjunto de valores (IN)  

**Ejemplo:**
```sql
SELECT customer_id, first_name
FROM customer
WHERE customer_id IN (
    SELECT customer_id 
    FROM payment 
    GROUP BY customer_id 
    HAVING SUM(amount) > 200
);
```

##### 7.4. Comparación de Rendimiento

**Problema:** Identificar clientes que han realizado más rentas que el promedio.

##### 7.4.1. Versión 1: Subconsulta Correlacionada (LENTO)

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

**Plan de ejecución:**
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
          ->  Index Only Scan using idx_fk_customer_id on rental r
Planning Time: 0.234 ms
Execution Time: 46.123 ms
```

**¿Por qué es lenta?**

Observe `loops=599` en `SubPlan 1`. La subconsulta se ejecutó **599 veces** (una por cada cliente).

**Problema fundamental:** Convierte una operación de conjunto en **N operaciones individuales**.

##### 7.4.2. Versión 2: JOIN con Tablas Derivadas (RÁPIDO)

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

**Plan de ejecución:**
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
  ->  Seq Scan on customer c  (actual time=0.012..0.234 rows=599 loops=1)
  ->  Hash  (actual time=7.890..7.890 rows=599 loops=1)
        ->  Seq Scan on rental  (actual time=0.012..3.456 rows=16044 loops=1)
Planning Time: 0.456 ms
Execution Time: 10.890 ms
```

**¿Por qué es rápida?**

Todos los valores de `loops=1`. Cada operación se ejecuta **una sola vez**.

##### 7.4.3. Comparación Directa

| Métrica | Subconsulta Correlacionada | JOIN con Tablas Derivadas | Mejora |
|---------|---------------------------|---------------------------|--------|
| **Execution Time** | 46.123 ms | 10.890 ms | **+76% más rápido** |
| **Escaneos de rental** | 599 veces | 1 vez | **99.8% menos** |
| **Operaciones de agregación** | 599 veces | 1 vez | **99.8% menos** |

**Nota sobre índices:** Los tiempos asumen que existe un índice en `rental.customer_id`:
```sql
CREATE INDEX idx_rental_customer_id ON rental(customer_id);
```

##### 7.4.4. Regla General

**Recomendación:** Siempre que sea posible, reescriba subconsultas correlacionadas como JOINs con tablas derivadas.

| Tamaño de Customer | Subconsulta Correlacionada | JOIN | Factor de Mejora |
|-------------------|---------------------------|------|------------------|
| 599 filas | 46 ms | 11 ms | 4x |
| 10,000 filas | ~800 ms | ~15 ms | 53x |
| 100,000 filas | ~8,000 ms | ~150 ms | 53x |
| 1,000,000 filas | ~80,000 ms | ~1,500 ms | 53x |

#### 8. Técnicas Avanzadas

##### 8.1. SELF JOIN (Auto-combinación)

Un **SELF JOIN** combina una tabla consigo misma.

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
    AND f1.film_id < f2.film_id
    AND f1.length != f2.length
WHERE f1.film_id <= 5
LIMIT 5;
```

**Análisis:** La condición `f1.film_id < f2.film_id` evita duplicados simétricos.

##### 8.2. LATERAL JOIN

Un **LATERAL JOIN** permite que la subconsulta del lado derecho acceda a columnas del lado izquierdo.

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

#### 9. Resumen de Conceptos Clave

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
| **CTE** | Conjunto nombrado | CTE = {...} | Conjunto temporal con nombre |

#### 10. Ejercicios Prácticos

##### Ejercicio 1: Ruta Geográfica Completa
Escriba una consulta que muestre:
- Nombre del cliente
- Dirección completa (calle, distrito, ciudad, país)
- Solo clientes activos
- Ordene por país y ciudad

**Pista:** Intersección sucesiva: Customer ∩ Address ∩ City ∩ Country

##### Ejercicio 2: Actores Prolíficos
Identifique los actores que han participado en más de 40 películas, mostrando:
- Nombre completo del actor
- Número de películas
- Lista de categorías (usar STRING_AGG)

**Pista:** Actor ⋈ Film_Actor ⋈ Film, GROUP BY y HAVING COUNT > 40

##### Ejercicio 3: Películas sin Alquiler
Encuentre las películas que **nunca** han sido rentadas.

**Pista:** Diferencia de conjuntos: Film − Rental (LEFT JOIN + WHERE IS NULL)

##### Ejercicio 4: Clientes por Encima del Promedio
Determine los clientes cuyo monto total de pagos excede el promedio global.

**Pista:** {c ∈ Customer | total(c) > avg(total)}

##### Ejercicio 5: Análisis con CTE
Genere un reporte de ventas mensuales con comparación contra el mes anterior usando CTEs.

**Pista:** Use LAG() dentro de un CTE para acceder al mes anterior.

#### 11. Recomendaciones de Buenas Prácticas

1. **Piense en términos de conjuntos:** Antes de escribir SQL, identifique qué operación de conjuntos necesita.

2. **Siempre use alias descriptivos:** `customer c` en lugar de solo `customer`.

3. **Califique todas las columnas:** `c.first_name` en lugar de `first_name`.

4. **Use EXISTS en lugar de IN** para subconsultas correlacionadas grandes.

5. **Evite SELECT * en JOINs:** Especifique solo las columnas necesarias.

6. **Índices en claves foráneas:** Asegúrese de que existan índices en las columnas de join.

7. **Analice el plan de ejecución:** Use `EXPLAIN ANALYZE` para optimizar consultas complejas.

8. **Use CTEs para lógica compleja:** Mejoran la legibilidad y mantenibilidad.

#### 12. Referencias

Codd, E. F. (1970). A relational model of data for large shared data banks. *Communications of the ACM*, *13*(6), 377–387. <https://doi.org/10.1145/362384.362685>

Date, C. J. (2003). *An introduction to database systems* (8th ed.). Addison-Wesley.

Gündüz, D. (s.f.). *Pagila: A sample PostgreSQL database* [Source code]. GitHub. <https://github.com/devrimgunduz/pagila>

PostgreSQL Global Development Group. (2026). *SQL SELECT: PostgreSQL 18.3 documentation*. <https://www.postgresql.org/docs/current/sql-select.html>

Wickham, H., Çetinkaya-Rundel, M., & Grolemund, G. (2023). *R for data science* (2nd. ed). O'Reilly Media. <https://r4ds.hadley.nz/>
