### Laboratorio 07: SQL en Tres Niveles: Básico, Intermedio y Avanzado

**Nivel:** Progresivo (tres módulos independientes de 60 minutos cada uno)  
**Duración total estimada:** 180 minutos (sesiones modulares)  
**Autor:** Dr. Jesús Zavala Ruiz  
**Fecha:** Mayo 2026  

**Entorno técnico:** Máquina virtual `fedora44-lab` (Fedora 44) + PostgreSQL 18 + Esquema `pagila`

---

> ADVERTENCIA ÉTICA:
> Esfuércese en realizar el laboratorio de manera sistemática y consciente con la finalidad de aprender, sin utilizar IA. La finalidad NO ES la estrega en sí de los entregables sino del desarrollo de conocimiento básico y habilidades de pensamiento reflexivo que lo/la lleven a reflexionar sobre el potencial y limitaciones que tienen las bases de datos relacionales, que seguirán siendo la espina dorsal de la operación de las organizaciones públicas y privadas.

**Si bien el laboratorio está pensado para desarrollar en tres fines de semana, es muy conveniente completarlo lo más pronto posible para comenzar a trabajar en el proyecto integrador.**

---

#### Fundamentación Pedagógica

Este laboratorio se estructura bajo un enfoque mayéutico: el conocimiento no se transmite como instrucción directa, sino que se construye mediante preguntas guía que inducen al estudiante a formular hipótesis, contrastar resultados y reflexionar sobre los principios subyacentes. Cada ejercicio depende conceptualmente del anterior, reforzando el aprendizaje mediante la reutilización deliberada de estructuras sintácticas en contextos de creciente complejidad.

El progreso escalonado responde a tres principios:
1. **Reactivación:** Cada fase inicia con una pregunta que recupera conocimientos previos.
2. **Extensión:** La nueva complejidad se introduce como variación controlada de lo ya dominado.
3. **Consolidación:** La reflexión posterior exige articular lo ejecutado con los fundamentos teóricos del modelo relacional.

### NIVEL BÁSICO: Fundamentos de Proyección, Filtrado y Agregación
**Duración:** 60 minutos

#### Objetivo Específico
Al concluir este módulo, el estudiante será capaz de formular consultas SQL elementales que proyecten columnas, filtren tuplas mediante predicados booleanos, ordenen resultados y calculen métricas agregadas, comprendiendo el orden lógico de evaluación de las cláusulas fundamentales.

#### Preguntas de Activación Cognitiva (5 min)
Antes de ejecutar cualquier instrucción, responda brevemente:
1. ¿Qué diferencia existe entre seleccionar "todas las columnas" mediante `SELECT *` y proyectar únicamente las columnas necesarias?
2. Si una consulta incluye `WHERE`, `GROUP BY` y `ORDER BY`, ¿en qué orden lógico se evalúan estas cláusulas? Fundamente su respuesta.
3. ¿Por qué cree que `LIMIT` se aplica al final del procesamiento de una consulta?

#### Fase 1: Conexión y Primera Proyección (10 min)

**Propósito:** Establecer comunicación con el sistema gestor y ejecutar la consulta mínima que recupera información.

##### Instrucciones Guiadas
1. Conéctese al cliente interactivo:
   ```bash
   psql -h localhost -U alumno -d pagila
   ```

2. Antes de listar las tablas, formule una hipótesis: ¿Qué comando de `psql` le permitiría visualizar la estructura de una tabla sin consultar documentación externa?

3. Verifique su hipótesis explorando la tabla `film`:
   ```sql
   \d film
   ```

4. Ejecute su primera consulta de proyección:
   ```sql
   SELECT title, release_year, rating FROM film LIMIT 5;
   ```

**Pregunta de reflexión posterior:**
- ¿Qué ocurriría si omitiera la cláusula `LIMIT` en una tabla con miles de registros? ¿Cómo afecta esto la experiencia de exploración?

#### Fase 2: Filtrado Lógico y Ordenamiento (15 min)

**Propósito:** Restringir el conjunto de resultados mediante condiciones y modificar su presentación.

##### Instrucciones Progresivas
1. Partiendo de la consulta anterior, ¿cómo modificaría la instrucción para mostrar únicamente películas con tarifa de renta superior a 4.99?
   ```sql
   SELECT title, rental_rate 
   FROM film 
   WHERE rental_rate > 4.99 
   ORDER BY rental_rate DESC 
   LIMIT 5;
   ```

2. Ahora, ¿cómo localizaría títulos que contengan la palabra "DRAGON", sin importar si están en mayúsculas o minúsculas?
   ```sql
   SELECT title, length, rating 
   FROM film 
   WHERE title ILIKE '%dragon%' 
   ORDER BY length DESC;
   ```

3. Combine condiciones: ¿Qué películas tienen duración entre 100 y 120 minutos Y clasificación PG o PG-13?
   ```sql
   SELECT title, length 
   FROM film 
   WHERE length BETWEEN 100 AND 120 
     AND rating IN ('PG', 'PG-13') 
   ORDER BY title ASC 
   LIMIT 10;
   ```

**Preguntas de consolidación:**
- ¿Por qué `WHERE` se evalúa antes de `ORDER BY`? ¿Qué implicación tiene esto en el rendimiento de la consulta?
- ¿En qué caso práctico utilizaría `ILIKE` en lugar de `LIKE`? ¿Existe un costo asociado?

#### Fase 3: Agregación y Agrupamiento (15 min)

**Propósito:** Calcular métricas resumen y segmentarlas por categorías lógicas.

##### Instrucciones Dependientes
1. Utilizando lo aprendido sobre proyección, ¿cómo calcularía el número total de películas, su duración promedio y el rango de tarifas?
   ```sql
   SELECT 
     COUNT(*) AS total_peliculas,
     AVG(length) AS duracion_promedio,
     MIN(rental_rate) AS tarifa_minima,
     MAX(rental_rate) AS tarifa_maxima
   FROM film;
   ```

2. Ahora, ¿cómo desagregaría ese conteo total por clasificación MPAA?
   ```sql
   SELECT rating, COUNT(*) AS total 
   FROM film 
   GROUP BY rating 
   ORDER BY total DESC;
   ```

3. Finalmente, ¿cómo filtraría únicamente aquellas clasificaciones cuya duración promedio exceda 110 minutos?
   ```sql
   SELECT rating, AVG(length) AS duracion_promedio
   FROM film
   GROUP BY rating
   HAVING AVG(length) > 110
   ORDER BY duracion_promedio DESC;
   ```

**Preguntas de reflexión crítica:**
- ¿Cuál es la diferencia semántica entre filtrar con `WHERE` y filtrar con `HAVING`? Ilustre con un ejemplo de este ejercicio.
- Si omitiera `GROUP BY` en la segunda consulta, ¿qué error esperaría que retornara PostgreSQL? ¿Por qué?

#### Fase 4: Primera Unión Relacional (15 min)

**Propósito:** Combinar información distribuida en dos relaciones mediante claves de vinculación.

##### Instrucciones Guiadas por Preguntas
1. Observe la estructura de `film` y `language`. ¿Qué columna cree que permite relacionar ambas tablas? Verifique con `\d film` y `\d language`.

2. Formule una hipótesis: ¿Cómo recuperaría el título de cada película junto con el nombre de su idioma original?
   ```sql
   SELECT f.title, f.release_year, l.name AS idioma
   FROM film f
   INNER JOIN language l ON f.language_id = l.language_id
   LIMIT 10;
   ```

3. Extienda la lógica anterior: ¿Cómo listaría películas de la categoría "Action"?
   ```sql
   SELECT f.title, c.name AS categoria
   FROM film f
   INNER JOIN film_category fc ON f.film_id = fc.film_id
   INNER JOIN category c ON fc.category_id = c.category_id
   WHERE c.name = 'Action'
   ORDER BY f.title ASC
   LIMIT 8;
   ```

**Pregunta de síntesis:**
- Si una consulta `JOIN` devuelve más filas de las esperadas, ¿qué patrón relacional (uno-a-muchos, muchos-a-muchos) podría estar generando este efecto? Fundamente con un ejemplo del esquema `pagila`.

#### Fase 5: Validación y Cierre (5 min)

##### Consulta Integradora
Ejecute la siguiente instrucción y explique, línea por línea, el propósito de cada cláusula:
```sql
SELECT 
  'Películas PG-13 con duración > 120 min' AS metrica,
  COUNT(*) AS resultado
FROM film
WHERE rating = 'PG-13' AND length > 120;
```

##### Reflexión Final (responder por escrito)
1. ¿Por qué el orden lógico de evaluación de una consulta SQL es: `FROM` -> `WHERE` -> `GROUP BY` -> `HAVING` -> `SELECT` -> `ORDER BY` -> `LIMIT`?
2. Si tuviera que explicar a un compañero la diferencia entre `COUNT(*)` y `COUNT(columna)`, ¿qué ejemplo utilizaría?
3. ¿Qué ventaja ofrece usar alias de tabla (`f`, `l`, `c`) en consultas con múltiples uniones?

#### Entregable del Nivel Básico
- Archivo `<matricula>_lab07_nivel_basico.sql` con las 10 consultas ejecutadas, comentadas explicando el propósito de cada cláusula.
- Capturas de pantalla de las salidas de las consultas de las fases 3.3 y 4.2.
- Respuestas fundamentadas a las preguntas de reflexión (máx. 100 palabras cada una).
- Formato de entrega: Telegram.
- Fecha límite: Domingo 31 de mayo de 2026, 23:59 h.

---

### NIVEL INTERMEDIO: Subconsultas, Operaciones de Conjunto y Funciones de Ventana
**Duración:** 60 minutos

#### Objetivo Específico
Al concluir este módulo, el estudiante será capaz de formular consultas que empleen subconsultas correlacionadas, operaciones de conjunto y funciones de ventana, comprendiendo cómo estas extensiones amplían la expresividad del álgebra relacional sin violar sus principios fundamentales.

#### Preguntas de Activación Cognitiva (5 min)
Recupere conocimientos del nivel básico:
1. ¿Cómo localizaría películas cuya tarifa exceda el promedio global utilizando únicamente lo aprendido hasta ahora?
2. ¿Qué limitación identifica en el enfoque anterior que justifique el uso de subconsultas?
3. Si deseara calcular un total acumulado por cliente, ¿por qué `GROUP BY` no sería suficiente?

#### Fase 1: Subconsultas – Expresividad mediante Anidamiento (15 min)

**Propósito:** Comprender cómo las subconsultas permiten encapsular lógica de filtrado y cálculo.

##### Instrucciones Progresivas
1. Partiendo de su respuesta a la pregunta de activación, formule la consulta que localice películas con tarifa superior al promedio:
   ```sql
   SELECT title, rental_rate 
   FROM film 
   WHERE rental_rate > (SELECT AVG(rental_rate) FROM film)
   ORDER BY rental_rate DESC 
   LIMIT 5;
   ```
   **Pregunta guía:** ¿Por qué PostgreSQL evalúa primero la subconsulta? ¿Qué tipo de resultado debe retornar para ser válido en este contexto?

2. Ahora, identifique clientes que nunca han realizado un pago. ¿Qué operador lógico le permitiría expresar "ausencia de coincidencia"?
   ```sql
   SELECT customer_id, first_name, last_name, email
   FROM customer c
   WHERE NOT EXISTS (
       SELECT 1 FROM payment p WHERE p.customer_id = c.customer_id
   )
   LIMIT 5;
   ```
   **Pregunta de contraste:** ¿En qué escenario `NOT IN` fallaría donde `NOT EXISTS` funciona correctamente? Considere el manejo de valores nulos.

3. Para cada película, muestre su duración y la duración promedio de su categoría. ¿Cómo estructuraría una subconsulta que dependa de la fila externa?
   ```sql
   SELECT 
       f.title,
       f.length,
       (SELECT AVG(f2.length) 
        FROM film f2
        JOIN film_category fc2 ON f2.film_id = fc2.film_id
        WHERE fc2.category_id = fc.category_id) AS avg_category_length
   FROM film f
   JOIN film_category fc ON f.film_id = fc.film_id
   LIMIT 8;
   ```
   **Pregunta de rendimiento:** ¿Por qué las subconsultas correlacionadas pueden impactar negativamente el tiempo de ejecución? ¿Qué alternativa podría optimizar esta lógica?

#### Fase 2: Operaciones de Conjunto y Uniones Avanzadas (10 min)

**Propósito:** Aplicar álgebra de conjuntos y explorar uniones que preservan filas no coincidentes.

##### Instrucciones Dependientes
1. Utilizando `LEFT JOIN`, liste todas las películas y su cantidad de copias en inventario, incluyendo aquellas sin stock. ¿Por qué `INNER JOIN` no sería adecuado aquí?
   ```sql
   SELECT f.title, COUNT(i.inventory_id) AS unidades_stock
   FROM film f
   LEFT JOIN inventory i ON f.film_id = i.film_id
   GROUP BY f.film_id, f.title
   ORDER BY unidades_stock ASC
   LIMIT 6;
   ```

2. Encuentre pares de actores que hayan actuado juntos. ¿Cómo utilizaría una tabla consigo misma para modelar esta relación reflexiva?
   ```sql
   SELECT DISTINCT
       a1.first_name || ' ' || a1.last_name AS actor_1,
       a2.first_name || ' ' || a2.last_name AS actor_2,
       f.title AS pelicula_compartida
   FROM film_actor fa1
   JOIN film_actor fa2 ON fa1.film_id = fa2.film_id AND fa1.actor_id < fa2.actor_id
   JOIN actor a1 ON fa1.actor_id = a1.actor_id
   JOIN actor a2 ON fa2.actor_id = a2.actor_id
   JOIN film f ON fa1.film_id = f.film_id
   LIMIT 5;
   ```

3. Liste películas de categoría "Action" que NO estén clasificadas como "NC-17". ¿Qué operador de conjunto expresaría esta diferencia?
   ```sql
   SELECT f.title FROM film f
   JOIN film_category fc ON f.film_id = fc.film_id
   JOIN category c ON fc.category_id = c.category_id
   WHERE c.name = 'Action'
   EXCEPT
   SELECT title FROM film WHERE rating = 'NC-17'
   ORDER BY 1 LIMIT 5;
   ```

**Pregunta de síntesis:**
- ¿Qué requisitos de compatibilidad deben cumplir las consultas combinadas con `UNION`, `INTERSECT` o `EXCEPT`? ¿Por qué existe esta restricción?

#### Fase 3: Funciones de Ventana – Análisis sin Colapso de Filas (15 min)

**Propósito:** Introducir cálculos analíticos que preservan la granularidad de cada tupla.

##### Instrucciones Guiadas
1. Clasifique películas por tarifa dentro de cada clasificación MPAA. ¿Cómo calcularía un ranking sin agrupar ni perder detalle?
   ```sql
   SELECT 
       title, rating, rental_rate,
       ROW_NUMBER() OVER(PARTITION BY rating ORDER BY rental_rate DESC) AS row_num,
       RANK() OVER(PARTITION BY rating ORDER BY rental_rate DESC) AS rank_num
   FROM film
   WHERE rating = 'PG'
   ORDER BY rental_rate DESC
   LIMIT 6;
   ```
   **Pregunta conceptual:** ¿En qué difiere `ROW_NUMBER()` de `RANK()` cuando existen empates en el criterio de ordenamiento?

2. Calcule el total gastado progresivamente por cada cliente. ¿Cómo acumularía valores sin colapsar las filas individuales?
   ```sql
   SELECT 
       customer_id, payment_date, amount,
       SUM(amount) OVER(PARTITION BY customer_id ORDER BY payment_date) AS running_total
   FROM payment
   WHERE customer_id = 1
   ORDER BY payment_date
   LIMIT 5;
   ```

3. Compare el monto de cada pago con el anterior del mismo cliente. ¿Qué función le permitiría acceder a una fila previa dentro de la misma partición?
   ```sql
   SELECT 
       customer_id, payment_date, amount,
       LAG(amount, 1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS prev_amount,
       amount - LAG(amount, 1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS diff
   FROM payment
   WHERE customer_id = 2
   ORDER BY payment_date
   LIMIT 5;
   ```

**Pregunta de reflexión crítica:**
- ¿Por qué las funciones de ventana no reemplazan a `GROUP BY`, sino que lo complementan? Proporcione un escenario donde ambas sean necesarias en la misma consulta.

#### Fase 4: Control Transaccional Básico (15 min)

**Propósito:** Ejecutar modificaciones atómicas dentro de bloques controlados.

##### Instrucciones con Puntos de Verificación
*Nota de seguridad: Todas las modificaciones se realizan sobre tablas temporales y se revierten.*

1. Inicie una transacción y cree una tabla temporal para registro de auditoría. ¿Qué instrucción garantiza que los cambios no sean permanentes hasta confirmación explícita?
   ```sql
   BEGIN;
   CREATE TEMP TABLE audit_log (
       id SERIAL PRIMARY KEY,
       action TEXT,
       timestamp TIMESTAMP DEFAULT NOW()
   );
   INSERT INTO audit_log (action) VALUES ('Inicio de actualización masiva');
   ```

2. Establezca un punto de guardado antes de una operación condicional. ¿Qué ventaja ofrece `SAVEPOINT` sobre un `ROLLBACK` completo?
   ```sql
   SAVEPOINT sp_before_update;
   INSERT INTO audit_log (action) VALUES ('Simulación de actualización de tarifas');
   UPDATE film SET rental_rate = rental_rate * 1.15 WHERE rating = 'R';
   ```

3. Verifique el impacto y, si excede expectativas, revierta únicamente la última operación:
   ```sql
   SELECT COUNT(*) AS films_updated FROM film WHERE rating = 'R' AND rental_rate > 4.99;
   ROLLBACK TO sp_before_update;
   ```

4. Confirme los cambios válidos y limpie el entorno:
   ```sql
   INSERT INTO audit_log (action) VALUES ('Transacción finalizada con éxito');
   COMMIT;
   SELECT * FROM audit_log;
   DROP TABLE audit_log;
   ```

**Pregunta de fundamentación teórica:**
- ¿Qué propiedad ACID garantiza que, ante un fallo durante `COMMIT`, los datos no queden en estado inconsistente? ¿Qué mecanismo de PostgreSQL implementa esta garantía?

#### Fase 5: Validación y Cierre (5 min)

##### Consulta Integradora
Ejecute y explique la siguiente consulta que combina subconsulta, ventana y filtro:
```sql
WITH ranked_payments AS (
    SELECT 
        customer_id, amount, payment_date,
        RANK() OVER(PARTITION BY customer_id ORDER BY amount DESC) AS rnk
    FROM payment
)
SELECT customer_id, amount, payment_date
FROM ranked_payments
WHERE rnk = 1
ORDER BY amount DESC
LIMIT 5;
```

##### Reflexión Final
1. ¿Por qué una expresión de tabla común (CTE) mejora la legibilidad frente a subconsultas anidadas profundas?
2. Si `EXPLAIN ANALYZE` muestra un `Seq Scan` en una columna frecuentemente filtrada, ¿qué acción de optimización consideraría?
3. ¿Cómo garantiza el principio de cierre relacional que el resultado de una consulta con funciones de ventana siga siendo una relación válida?

#### Entregable del Nivel Intermedio
- Archivo `<matr'icula>_lab07_nivel_intermedio.sql` con las 12 consultas ejecutadas, comentadas explicando el rol de cada cláusula avanzada.
- Capturas de pantalla de las salidas de las consultas 1.3, 3.2 y 4.4.
- Respuestas fundamentadas a las preguntas de reflexión (máx. 120 palabras cada una).
- Formato de entrega: Telegram.
- Fecha límite: Domingo 7 de junio de 2026, 23:59 h.

---

### NIVEL AVANZADO: CTEs, Funciones Analíticas, DML Transaccional y Optimización
**Duración:** 60 minutos

#### Objetivo Específico
Al concluir este módulo, el estudiante será capaz de diseñar pipelines de consulta complejos mediante CTEs, aplicar funciones de ventana analíticas con marcos explícitos, gestionar actualizaciones atómicas con cláusulas `RETURNING` y `ON CONFLICT`, e interpretar planes de ejecución para fundamentar decisiones de optimización.

>Nota:
> Una **Expresión de Tabla Común** (del inglés **Common Table Expression**, **CTE**) es una construcción sintáctica de SQL que permite definir un resultado temporal con nombre, el cual puede ser referenciado dentro de una consulta SELECT, INSERT, UPDATE, DELETE o MERGE. Su declaración se realiza mediante la cláusula WITH, previa a la sentencia principal.
> Según Date (2019), las CTEs materializan el principio de cierre relacional: el resultado de toda consulta es una relación, y por tanto puede servir como operando de otra consulta. Las CTEs hacen explícita esta propiedad mediante una sintaxis que favorece la legibilidad y la descomposición lógica de problemas complejos.

#### Preguntas de Activación Cognitiva (5 min)
Recupere y conecte conocimientos previos:
1. ¿Cómo estructuraría una consulta que requiera dos niveles de agregación (por tienda y por mes) sin anidamiento excesivo?
2. Si deseara calcular un promedio móvil de 7 días, ¿por qué una función de ventana con marco explícito sería preferible a una subconsulta correlacionada?
3. ¿Qué mecanismo de PostgreSQL permitiría insertar o actualizar un registro en una sola instrucción, evitando condiciones de carrera?

#### Fase 1: CTEs – Pipelines Lógicos y Descomposición de Problemas (15 min)

**Propósito:** Utilizar `WITH` para estructurar consultas complejas en bloques semánticos reutilizables.

##### Instrucciones Progresivas
1. Analice la rentabilidad por tienda. ¿Cómo descompondría este problema en dos etapas: cálculo de ingresos y ranking posterior?
   ```sql
   WITH store_revenue AS (
       SELECT 
           s.store_id,
           c.city,
           SUM(p.amount) AS total_revenue,
           COUNT(DISTINCT p.customer_id) AS active_customers
       FROM store s
       JOIN address a ON s.address_id = a.address_id
       JOIN city c ON a.city_id = c.city_id
       JOIN customer cu ON s.store_id = cu.store_id
       JOIN payment p ON cu.customer_id = p.customer_id
       GROUP BY s.store_id, c.city
   ),
   revenue_rank AS (
       SELECT 
           store_id, city, total_revenue, active_customers,
           RANK() OVER(ORDER BY total_revenue DESC) AS revenue_rank
       FROM store_revenue
   )
   SELECT store_id, city, total_revenue, active_customers, revenue_rank
   FROM revenue_rank
   WHERE revenue_rank <= 2;
   ```
   **Pregunta guía:** ¿Qué ventaja ofrece nombrar explícitamente cada CTE frente a anidar subconsultas?

2. Filtre películas con tarifa superior al promedio antes de agrupar por categoría. ¿Cómo evitaría recalcular el promedio en cada fila?
   ```sql
   WITH high_value_films AS (
       SELECT film_id, title, rental_rate
       FROM film
       WHERE rental_rate > (SELECT AVG(rental_rate) FROM film)
   )
   SELECT 
       c.name AS category,
       COUNT(hvf.film_id) AS films_above_avg_price,
       AVG(hvf.rental_rate) AS avg_premium_rate
   FROM high_value_films hvf
   JOIN film_category fc ON hvf.film_id = fc.film_id
   JOIN category c ON fc.category_id = c.category_id
   GROUP BY c.name
   ORDER BY films_above_avg_price DESC
   LIMIT 5;
   ```
   **Pregunta de optimización:** ¿En qué circunstancias el optimizador de PostgreSQL podría "inlinear" un CTE para evitar materialización intermedia?

#### Fase 2: Funciones de Ventana Avanzadas – Marcos Explícitos y Segmentación (15 min)

**Propósito:** Extender el análisis secuencial y distribucional mediante operadores de ventana con control preciso del marco de cálculo.

##### Instrucciones Dependientes
1. Analice la brecha temporal entre pagos consecutivos de un cliente. ¿Cómo accedería simultáneamente al pago anterior y siguiente?
   ```sql
   SELECT 
       customer_id, payment_date, amount,
       LAG(amount, 1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS prev_amount,
       LEAD(amount, 1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS next_amount,
       amount - LAG(amount, 1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS diff_from_prev
   FROM payment
   WHERE customer_id = 2
   ORDER BY payment_date
   LIMIT 6;
   ```

2. Segmenté clientes en cuartiles según gasto total. ¿Cómo combinaría `NTILE` con `PERCENT_RANK` para obtener una visión dual de la distribución?
   ```sql
   WITH customer_spend AS (
       SELECT 
           customer_id,
           SUM(amount) AS total_spent
       FROM payment
       GROUP BY customer_id
   )
   SELECT 
       customer_id, total_spent,
       NTILE(4) OVER(ORDER BY total_spent DESC) AS spend_quartile,
       PERCENT_RANK() OVER(ORDER BY total_spent DESC) AS percentile_rank
   FROM customer_spend
   ORDER BY total_spent DESC
   LIMIT 8;
   ```

3. Calcule un promedio móvil de ingresos diarios. ¿Cómo definiría explícitamente el marco de 7 días precedentes?
   ```sql
   SELECT 
       payment_date::date AS dia,
       SUM(amount) AS daily_revenue,
       SUM(SUM(amount)) OVER(
           ORDER BY payment_date::date 
           ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
       ) AS rolling_7day_total
   FROM payment
   WHERE payment_date >= '2007-04-01' AND payment_date < '2007-05-01'
   GROUP BY payment_date::date
   ORDER BY dia
   LIMIT 7;
   ```

**Pregunta de fundamentación teórica:**
- ¿Por qué la cláusula `ROWS BETWEEN` es crucial para cálculos de promedios móviles? ¿Qué ocurriría si se omitiera y se usara el marco por defecto?

#### Fase 3: DML Transaccional Avanzado – UPSERT y Control de Concurrencia (15 min)

**Propósito:** Ejecutar modificaciones atómicas con manejo explícito de conflictos y retorno de metadatos.

##### Instrucciones con Puntos de Decisión
*Nota de seguridad: Todas las operaciones se realizan sobre tablas temporales o se revierten.*

1. Inserte nuevos clientes o actualice sus datos si ya existen. ¿Qué cláusula evitaría una verificación previa en la aplicación?
   ```sql
   BEGIN;
   CREATE TEMP TABLE new_customers (
       first_name VARCHAR(45), last_name VARCHAR(45), 
       email VARCHAR(50), address_id SMALLINT
   );
   INSERT INTO new_customers VALUES 
       ('ANA', 'LOPEZ', 'ana.lopez@email.com', 1),
       ('CARLOS', 'RUIZ', 'carlos.ruiz@email.com', 1),
       ('PENELOPE', 'GUINESS', 'penelope.guiness@email.com', 1);

   INSERT INTO customer (first_name, last_name, email, address_id, activebool)
   SELECT first_name, last_name, email, address_id, true
   FROM new_customers
   ON CONFLICT (email) DO UPDATE SET
       first_name = EXCLUDED.first_name,
       last_name = EXCLUDED.last_name,
       activebool = true
   RETURNING customer_id, email, xmin AS transaction_id;
   ```
   **Pregunta conceptual:** ¿Qué ventaja ofrece `RETURNING` frente a ejecutar una consulta `SELECT` posterior para verificar los cambios?

2. Establezca un punto de guardado antes de una actualización masiva condicional. ¿Cómo revertiría únicamente esta operación sin afectar inserciones previas?
   ```sql
   SAVEPOINT sp_before_inventory_update;
   
   UPDATE inventory 
   SET last_update = NOW()
   WHERE film_id IN (
       SELECT film_id FROM film WHERE rating = 'NC-17' AND length > 150
   );
   
   SELECT COUNT(*) AS updated_rows FROM inventory 
   WHERE last_update > NOW() - INTERVAL '10 seconds';
   
   -- ROLLBACK TO sp_before_inventory_update; -- Descomentar si se requiere revertir
   
   COMMIT;
   ```

**Pregunta de reflexión crítica:**
- Si un `UPSERT` falla por conflicto de concurrencia (serialización), ¿qué patrón de reintento con retroceso exponencial implementaría en una aplicación productiva?

#### Fase 4: Planes de Ejecución – Conciencia de Optimización (10 min)

**Propósito:** Interpretar `EXPLAIN (ANALYZE, BUFFERS)` para identificar cuellos de botella y comprender la transformación de consultas declarativas en planes físicos.

##### Instrucciones Guiadas por Preguntas
1. Analice el plan de una consulta que une tres tablas con filtro por apellido. ¿Qué estrategia de unión selecciona el optimizador y por qué?
   ```sql
   EXPLAIN (ANALYZE, BUFFERS) 
   SELECT f.title, a.first_name, a.last_name
   FROM film f
   JOIN film_actor fa ON f.film_id = fa.film_id
   JOIN actor a ON fa.actor_id = a.actor_id
   WHERE a.last_name = 'GUINESS';
   ```

2. Evalúe el impacto de un filtro selectivo en una consulta agregada. ¿Cómo cambia la estrategia de acceso a datos?
   ```sql
   EXPLAIN (ANALYZE, BUFFERS)
   SELECT c.city, COUNT(p.payment_id)
   FROM payment p
   JOIN customer cu ON p.customer_id = cu.customer_id
   JOIN address a ON cu.address_id = a.address_id
   JOIN city c ON a.city_id = c.city_id
   WHERE p.amount > 9.99
   GROUP BY c.city;
   ```

**Guía de interpretación (buscar en la salida):**
- `Seq Scan` vs `Index Scan` / `Bitmap Index Scan`: ¿Cuándo es preferible cada uno?
- `Nested Loop` vs `Hash Join` vs `Merge Join`: ¿Qué factores determinan la selección del optimizador?
- `rows=... loops=...`: ¿Cómo interpretar la discrepancia entre cardinalidad estimada y real?
- `Buffers: shared hit=... read=...`: ¿Qué indica sobre el uso de caché versus acceso a disco?

**Pregunta de síntesis teórica:**
- Date (2019) afirma que "la independencia física depende de que el optimizador pueda transformar equivalentes algebraicos sin alterar semántica". ¿Cómo se manifiesta este principio en la diferencia entre una consulta escrita por un humano y el plan físico generado por PostgreSQL?

#### Fase 5: Validación Integradora y Reflexión Final (5 min)

##### Consulta de Síntesis
Ejecute y explique la siguiente consulta que combina CTE, ventana y análisis temporal:
```sql
WITH monthly_store AS (
    SELECT 
        DATE_TRUNC('month', p.payment_date) AS mes,
        s.store_id,
        SUM(p.amount) AS revenue
    FROM payment p
    JOIN customer cu ON p.customer_id = cu.customer_id
    JOIN store s ON cu.store_id = s.store_id
    GROUP BY DATE_TRUNC('month', p.payment_date), s.store_id
)
SELECT 
    mes, store_id, revenue,
    LAG(revenue, 1) OVER(PARTITION BY store_id ORDER BY mes) AS prev_month_revenue,
    revenue - LAG(revenue, 1) OVER(PARTITION BY store_id ORDER BY mes) AS mom_change
FROM monthly_store
WHERE mes >= '2007-03-01' AND mes < '2007-06-01'
ORDER BY store_id, mes;
```

##### Reflexión Final del Laboratorio Completo
1. ¿Cómo garantiza el principio de cierre relacional que un CTE pueda alimentarse a otro sin violar la independencia lógica entre niveles de abstracción?
2. Si `EXPLAIN ANALYZE` ejecuta la consulta, ¿qué implicaciones tiene esto en entornos productivos con tablas de gran volumen? ¿Qué precauciones tomaría?
3. ¿Qué mecanismo de control de concurrencia multiversión (MVCC) de PostgreSQL previene lecturas sucias durante operaciones `UPSERT` simultáneas? ¿Cómo se refleja esto en el sistema de identificadores de transacción (`xmin`)?

#### Entregable del Nivel Avanzado
- Archivo `<matricula>_lab07_nivel_avanzado.sql` con las 8 consultas ejecutadas, comentadas explicando el rol de cada cláusula avanzada.
- Capturas de pantalla de las salidas de las consultas 1.1, 2.2 y 3.1.
- Respuestas fundamentadas a las preguntas de reflexión final (máx. 120 palabras cada una).
- Formato de entrega: Telegram
- Fecha límite: Domingo 14 de junio de 2026, 23:59 h.

---

#### Rúbrica de Evaluación Unificada (aplicable a los tres niveles)

| Criterio | Ponderación | Indicadores de Desempeño |
|----------|-------------|-------------------------|
| Sintaxis correcta y precisa | 30% | Ausencia de errores de parsing; uso adecuado de comas, alias y cláusulas según el nivel |
| Propósito semántico documentado | 25% | Comentarios explican claramente el rol de cada cláusula y su contribución al resultado |
| Consistencia de resultados | 20% | Salidas coinciden con la estructura y datos reales del esquema `pagila` |
| Reflexión conceptual fundamentada | 15% | Respuestas demuestran comprensión teórica, diferenciación clara entre constructores y principios subyacentes |
| Formato y documentación | 10% | Archivo legible, nombrado correctamente, entregado en plazo y con estructura clara |

---

#### Consideraciones de Seguridad y Buenas Prácticas

- Todas las consultas de los niveles básico e intermedio son de solo lectura (`SELECT`). Las operaciones de escritura del nivel avanzado se ejecutan dentro de transacciones explícitas y se aíslan en tablas temporales o se revierten. **No se modifican tablas base permanentemente en entornos académicos.**
- Evite el uso de `SELECT *` en consultas productivas; proyecte únicamente las columnas necesarias para reducir consumo de ancho de banda y memoria.
- Verifique siempre el plan de ejecución (`EXPLAIN ANALYZE`) en consultas complejas para identificar oportunidades de optimización mediante índices o reescritura lógica.
- Mantenga la sesión de `psql` activa únicamente durante la práctica; cierre la conexión con `\q` al finalizar para liberar recursos del servidor.
- En entornos productivos, las modificaciones masivas deben programarse en ventanas de mantenimiento, respaldarse previamente mediante `pg_dump` o snapshots, y ejecutarse con roles de privilegio mínimo necesario.

---

#### Referencias Bibliográficas

- devrimgunduz. (2026). *Pagila - Sample Database for PostgreSQL*. GitHub. https://github.com/devrimgunduz/pagila
- Date, C. J. (2019). *Database Design and Relational Theory: Normal Forms and All That Jazz* (2ª ed.). Apress.
- Mata-Toledo, R. A., & Cushman, P. K. (2000). *Schaum's Outline of Fundamentals of Relational Databases*. McGraw-Hill.
- PostgreSQL Global Development Group. (2026). *PostgreSQL 18 Documentation: Query Syntax, Window Functions, Transaction Control, EXPLAIN*. <https://www.postgresql.org/docs/18/>

