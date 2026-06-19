### Lectura 06. Fundamentos del Lenguaje de Consulta Estructurada (SQL)

**Dr. Jesús Zavala Ruiz**

**Contacto:**
- **Correo electrónico:** [jzr@xanum.uam.mx](mailto:jzr@xanum.uam.mx)
- **Telegram:** <https://t.me/jzavalar>

Última actualización: 19 de junio de 2026.

---

#### 1. Introducción

El *lenguaje de consulta estructurado* o *SQL* (por sus siglas en inglés *Structured Query Language*) es el lenguaje estándar para acceder y manipular bases de datos relacionales. A diferencia de los lenguajes de programación procedurales como Python o Java, SQL es un lenguaje *declarativo* de cuarta generación (4GL): en lugar de especificar *cómo* obtener los datos paso a paso, se describe *qué* datos se desean obtener.

SQL se convirtió en estándar de la ANSI en 1986 y de la ISO en 1987, con revisiones continuas hasta SQL:2023. Aunque existe un estándar, cada sistema gestor de bases de datos (SGBD) implementa extensiones propias. En estas notas nos enfocaremos principalmente en **PostgreSQL**, un sistema de gestión de bases de datos relacional de código abierto que cumple rigurosamente con el estándar SQL y es ampliamente utilizado en la industria.

#### 1.1. ¿Qué es una base de datos relacional?

Una *base de datos relacional* organiza la información en *tablas* compuestas por *filas* (registros o tuplas) y *columnas* (atributos o campos). Las tablas se relacionan entre sí mediante *claves primarias* y *claves foráneas*, estableciendo vínculos lógicos entre los datos.

Por ejemplo, considérese una base de datos de una empresa con dos tablas:

```
┌─────────────────────────────────────────────────────────────┐
│ Tabla: departamentos                                        │
├─────────────────┬──────────────────────┬────────────────────┤
│ id_departamento │ nombre_departamento  │ ubicacion          │
├─────────────────┼──────────────────────┼────────────────────┤
│              10 │ Recursos Humanos     │ Edificio A, Piso 2 │
│              20 │ Tecnología           │ Edificio B, Piso 3 │
│              30 │ Ventas               │ Edificio A, Piso 1 │
└─────────────────┴──────────────────────┴────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ Tabla: empleados                                                        │
├──────────────┬──────────┬───────────┬──────────┬─────────────────┬────────┐
│ id_empleado  │ nombre   │ apellido  │ salario  │ departamento_id │ activo │
├──────────────┼──────────┼───────────┼──────────┼─────────────────┼────────┤
│            1 │ Juan     │ Pérez     │ 35000.00 │              10 │ TRUE   │
│            2 │ María    │ González  │ 42000.00 │              20 │ TRUE   │
│            3 │ Carlos   │ López     │ 38000.00 │              10 │ TRUE   │
│            4 │ Ana      │ Martínez  │ 45000.00 │              30 │ TRUE   │
└──────────────┴──────────┴───────────┴──────────┴─────────────────┴────────┘
```

La columna `departamento_id` en la tabla `empleados` es una *clave foránea* que referencia a `id_departamento` en la tabla `departamentos`, estableciendo una relación entre ambas tablas.

#### 1.2. Orientación a filas vs. orientación a columnas

Una distinción arquitectónica importante es cómo los SGBD almacenan físicamente los datos en disco:

**Bases de datos orientadas a filas** (PostgreSQL, MySQL, SQLite, SQL Server): Los valores de una fila completa se almacenan contiguamente.

```
Fila 1: [id=1, nombre='Ana', edad=25]   → almacenados juntos
Fila 2: [id=2, nombre='Luis', edad=30]  → almacenados juntos
```

**Bases de datos orientadas a columnas** (Redshift, BigQuery, Snowflake, Presto): Los valores de una columna se almacenan contiguamente.

```
Columna id:     [1, 2, 3]              → almacenados juntos
Columna nombre: ['Ana', 'Luis', 'María'] → almacenados juntos
Columna edad:   [25, 30, 22]           → almacenados juntos
```

| Característica | Orientado a filas | Orientado a columnas |
| :-: | :-: | :-: |
| Caso de uso | Transacciones (OLTP), servicios web | Análisis de datos (OLAP) |
| Ventaja | Rápido para consultar/actualizar filas completas | Rápido para agregar columnas completas |
| Desventaja | Consultas analíticas complejas son lentas | `SELECT *` puede ser muy lento |

PostgreSQL es orientado a filas, optimizado para aplicaciones transaccionales donde se necesitan leer y escribir registros completos frecuentemente.

#### 1.3. Sublenguajes de SQL

SQL se divide en varios sublenguajes según su propósito:

- **DDL (Data Definition Language):** Define la estructura de la base de datos (`CREATE`, `ALTER`, `DROP`)
- **DML (Data Manipulation Language):** Manipula los datos (`SELECT`, `INSERT`, `UPDATE`, `DELETE`)
- **DCL (Data Control Language):** Controla permisos y acceso (`GRANT`, `REVOKE`)
- **TCL (Transaction Control Language):** Gestiona transacciones (`BEGIN`, `COMMIT`, `ROLLBACK`)

---

#### 2. Preparación del entorno en PostgreSQL

#### 2.1. Instalación y conexión

PostgreSQL puede instalarse desde https://www.postgresql.org/download/. Una vez instalado, se accede mediante el cliente de línea de comandos `psql`:

```bash
# Conectar a la base de datos
psql -U postgres -h localhost -d empresa_db

# O simplemente
psql empresa_db
```

#### 2.2. Creación de la base de datos

```sql
-- Crear una base de datos
CREATE DATABASE empresa_db
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'es_MX.UTF-8'
    LC_CTYPE = 'es_MX.UTF-8';

-- Conectar a la base de datos (desde psql)
\c empresa_db
```

#### 2.3. Carga de datos desde CSV

Para los ejemplos de esta lectura, utilizaremos dos archivos CSV.

**Archivo `departamentos.csv`:**

```csv
id_departamento,nombre_departamento,ubicacion,presupuesto
10,Recursos Humanos,Edificio A Piso 2,500000.00
20,Tecnología,Edificio B Piso 3,1200000.00
30,Ventas,Edificio A Piso 1,800000.00
40,Marketing,Edificio C Piso 1,600000.00
```

**Archivo `empleados.csv`:**

```csv
id_empleado,nombre,apellido,fecha_nacimiento,salario,departamento_id,activo
1,Juan,Pérez,1990-05-15,35000.00,10,TRUE
2,María,González,1985-08-22,42000.00,20,TRUE
3,Carlos,López,1992-03-10,38000.00,10,TRUE
4,Ana,Martínez,1988-11-30,45000.00,30,TRUE
5,Pedro,Rodríguez,1995-07-04,32000.00,20,TRUE
6,Laura,Hernández,1991-01-18,41000.00,10,FALSE
7,Miguel,Sánchez,1987-09-25,48000.00,30,TRUE
8,Sofía,Ramírez,1993-12-12,36000.00,20,TRUE
9,Diego,Flores,1989-06-08,44000.00,10,TRUE
10,Elena,Torres,1994-04-20,33000.00,30,TRUE
```

**Creación de tablas y carga:**

```sql
-- Crear tabla de departamentos
CREATE TABLE departamentos (
    id_departamento INTEGER PRIMARY KEY,
    nombre_departamento VARCHAR(100) NOT NULL UNIQUE,
    ubicacion VARCHAR(200),
    presupuesto NUMERIC(12,2)
);

-- Crear tabla de empleados
CREATE TABLE empleados (
    id_empleado INTEGER PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    fecha_nacimiento DATE,
    salario NUMERIC(10,2) CHECK (salario > 0),
    departamento_id INTEGER,
    fecha_contratacion DATE DEFAULT CURRENT_DATE,
    activo BOOLEAN DEFAULT TRUE,
    CONSTRAINT fk_empleado_departamento
        FOREIGN KEY (departamento_id)
        REFERENCES departamentos(id_departamento)
);

-- Cargar datos desde CSV (desde psql)
\copy departamentos FROM 'departamentos.csv' WITH (FORMAT csv, HEADER true);
\copy empleados FROM 'empleados.csv' WITH (FORMAT csv, HEADER true);

-- Verificar la carga
SELECT COUNT(*) AS total_departamentos FROM departamentos;
SELECT COUNT(*) AS total_empleados FROM empleados;
```

---

#### 3. Elementos básicos del lenguaje SQL

#### 3.1. Comentarios

Los comentarios son texto que no se ejecuta y sirven para documentar el código:

```sql
-- Comentario de una sola línea

SELECT nombre FROM empleados; -- Comentario al final de la línea

/* Comentario
   de múltiples
   líneas */

/* Archivo: consulta_basica.sql
   Objetivo: Selecciona empleados activos
   Autor: Prof. J Zavala
   Fecha: 2026-06-20 */
```

#### 3.2. Identificadores

Un *identificador* nombra objetos de la base de datos (tablas, columnas, vistas, etc.). Reglas básicas en PostgreSQL:

- Consta de letras, dígitos (0-9) y guión bajo (`_`)
- Debe comenzar con una letra o guión bajo
- Los identificadores no delimitados no distinguen mayúsculas/minúsculas
- Los identificadores delimitados (entre comillas dobles `"nombre"`) sí distinguen

**Buena práctica:** usar minúsculas con guiones bajos: `id_empleado`, `fecha_nacimiento`, `salario_mensual`.

#### 3.3. Literales

Los *literales* son valores fijos escritos directamente en las consultas:

```sql
-- Literales numéricos
42
-17
3.14159
100.50

-- Literales de cadena (apóstrofes simples)
'Hola Mundo'
'Juan Pérez'
'2026-06-20'

-- Literales de fecha y hora
DATE '2026-06-20'
TIME '14:30:00'
TIMESTAMP '2026-06-20 14:30:00'

-- Literales booleanos
TRUE
FALSE

-- Literal nulo
NULL
```

#### 3.4. Tipos de datos en PostgreSQL

PostgreSQL ofrece una rica variedad de tipos de datos:

**Numéricos:**

| Tipo | Descripción | Ejemplo |
| :-: | :-- | :-: |
| `SMALLINT` | Entero pequeño (2 bytes) | `32767` |
| `INTEGER` | Entero (4 bytes) | `2147483647` |
| `BIGINT` | Entero grande (8 bytes) | `9223372036854775807` |
| `NUMERIC(p,s)` | Decimal exacto | `NUMERIC(10,2)` → `12345678.90` |
| `REAL` | Punto flotante simple precisión | `3.14` |
| `DOUBLE PRECISION` | Punto flotante doble precisión | `3.14159265358979` |
| `SERIAL` | Entero autoincremental | `1, 2, 3, ...` |

**Cadenas de caracteres:**

| Tipo | Descripción | Ejemplo |
| :-: | :-- | :-: |
| `CHAR(n)` | Longitud fija | `CHAR(10)` → `'Hola      '` |
| `VARCHAR(n)` | Longitud variable hasta n | `VARCHAR(50)` → `'Juan'` |
| `TEXT` | Longitud ilimitada | `'Texto muy largo...'` |

**Fecha y hora:**

| Tipo | Descripción | Ejemplo |
| :-: | :-- | :-: |
| `DATE` | Fecha | `DATE '2026-06-20'` |
| `TIME` | Hora | `TIME '14:30:00'` |
| `TIMESTAMP` | Fecha y hora | `TIMESTAMP '2026-06-20 14:30:00'` |
| `INTERVAL` | Intervalo de tiempo | `INTERVAL '1 year 2 months'` |

**Booleano:**

| Tipo | Descripción | Valores |
| :-: | :-- | :-: |
| `BOOLEAN` | Valor lógico | `TRUE`, `FALSE`, `NULL` |

**Otros tipos importantes:**

- `JSON` / `JSONB`: Datos JSON
- `ARRAY`: Arreglos de cualquier tipo
- `UUID`: Identificadores únicos universales
- `BYTEA`: Datos binarios

---

#### 4. Anatomía de una consulta SELECT

Una de las confusiones más frecuentes al aprender SQL es la diferencia entre el *orden de escritura* y el *orden de ejecución* de una consulta.

#### 4.1. Orden de escritura

La sintaxis de `SELECT` debe escribirse en este orden estricto:

```sql
SELECT <columnas_o_expresiones>      -- (6) Proyección final
FROM <tabla_o_vista>                 -- (1) Tabla origen
[WHERE <condición>]                  -- (2) Filtrado de filas
[GROUP BY <columnas>]                -- (3) Agrupamiento
[HAVING <condición_de_grupo>]        -- (4) Filtrado de grupos
[ORDER BY <columnas>]                -- (7) Ordenamiento
[LIMIT <número>];                    -- (8) Limitación
```

#### 4.2. Orden de ejecución lógico

El motor de PostgreSQL procesa la consulta en este orden:

| Paso | Cláusula | Operación |
| :-: | :-: | :-- |
| 1 | `FROM` | Identifica tabla(s) origen y realiza `JOIN` |
| 2 | `WHERE` | Filtra filas que no cumplen la condición |
| 3 | `GROUP BY` | Agrupa filas por valores de columnas |
| 4 | `HAVING` | Filtra grupos que no cumplen la condición |
| 5 | `SELECT` | Proyecta columnas y expresiones finales |
| 6 | `ORDER BY` | Ordena el resultado |
| 7 | `LIMIT` | Recorta el número de filas |

#### 4.3. Ilustración paso a paso

Considérese esta consulta:

```sql
SELECT departamento_id, COUNT(*) AS total_empleados
FROM empleados
WHERE activo = TRUE
GROUP BY departamento_id
HAVING COUNT(*) >= 2
ORDER BY total_empleados DESC;
```

**Paso 1 — `FROM empleados`:** se obtiene la tabla completa.

```
┌─────────────┬────────┬───────────┬──────────┬─────────────────┬────────┐
│ id_empleado │ nombre │ apellido  │ salario  │ departamento_id │ activo │
├─────────────┼────────┼───────────┼──────────┼─────────────────┼────────┤
│           1 │ Juan   │ Pérez     │ 35000.00 │              10 │ TRUE   │
│           2 │ María  │ González  │ 42000.00 │              20 │ TRUE   │
│           3 │ Carlos │ López     │ 38000.00 │              10 │ TRUE   │
│           4 │ Ana    │ Martínez  │ 45000.00 │              30 │ TRUE   │
│           5 │ Pedro  │ Rodríguez │ 32000.00 │              20 │ TRUE   │
│           6 │ Laura  │ Hernández │ 41000.00 │              10 │ FALSE  │
│           7 │ Miguel │ Sánchez   │ 48000.00 │              30 │ TRUE   │
│           8 │ Sofía  │ Ramírez   │ 36000.00 │              20 │ TRUE   │
│           9 │ Diego  │ Flores    │ 44000.00 │              10 │ TRUE   │
│          10 │ Elena  │ Torres    │ 33000.00 │              30 │ TRUE   │
└─────────────┴────────┴───────────┴──────────┴─────────────────┴────────┘
```

**Paso 2 — `WHERE activo = TRUE`:** se descartan las filas que no cumplen.

```
┌─────────────┬────────┬───────────┬──────────┬─────────────────┬────────┐
│ id_empleado │ nombre │ apellido  │ salario  │ departamento_id │ activo │
├─────────────┼────────┼───────────┼──────────┼─────────────────┼────────┤
│           1 │ Juan   │ Pérez     │ 35000.00 │              10 │ TRUE   │
│           2 │ María  │ González  │ 42000.00 │              20 │ TRUE   │
│           3 │ Carlos │ López     │ 38000.00 │              10 │ TRUE   │
│           4 │ Ana    │ Martínez  │ 45000.00 │              30 │ TRUE   │
│           5 │ Pedro  │ Rodríguez │ 32000.00 │              20 │ TRUE   │
│           7 │ Miguel │ Sánchez   │ 48000.00 │              30 │ TRUE   │
│           8 │ Sofía  │ Ramírez   │ 36000.00 │              20 │ TRUE   │
│           9 │ Diego  │ Flores    │ 44000.00 │              10 │ TRUE   │
│          10 │ Elena  │ Torres    │ 33000.00 │              30 │ TRUE   │
└─────────────┴────────┴───────────┴──────────┴─────────────────┴────────┘
(Laura Hernández, id=6, eliminada)
```

**Paso 3 — `GROUP BY departamento_id`:** se agrupan las filas.

```
┌─────────────────┬──────────────────────────────────────────────┐
│ departamento_id │ Filas en el grupo                            │
├─────────────────┼──────────────────────────────────────────────┤
│              10 │ Juan Pérez, Carlos López, Diego Flores       │
│              20 │ María González, Pedro Rodríguez, Sofía Ramírez│
│              30 │ Ana Martínez, Miguel Sánchez, Elena Torres   │
└─────────────────┴──────────────────────────────────────────────┘
```

**Paso 4 — `HAVING COUNT(*) >= 2`:** se filtran los grupos. Todos tienen 3 miembros, así que ninguno se descarta.

**Paso 5 — `SELECT departamento_id, COUNT(*) AS total_empleados`:** se proyectan las columnas finales.

```
┌─────────────────┬─────────────────┐
│ departamento_id │ total_empleados │
├─────────────────┼─────────────────┤
│              10 │               3 │
│              20 │               3 │
│              30 │               3 │
└─────────────────┴─────────────────┘
```

**Paso 6 — `ORDER BY total_empleados DESC`:** se ordena. Como todos los valores son iguales, el orden puede variar.

**Importante:** esta comprensión explica por qué no se puede usar un alias de `SELECT` en `WHERE`: el `WHERE` se ejecuta antes de que el alias exista.

---

#### 5. Consulta básica de datos

#### 5.1. Selección de columnas

```sql
-- Seleccionar columnas específicas
SELECT nombre, apellido, salario
FROM empleados;

-- Seleccionar todas las columnas
SELECT *
FROM departamentos;

-- Usar alias para columnas
SELECT nombre AS nombre_empleado,
       apellido AS apellido_empleado,
       salario AS salario_mensual
FROM empleados;

-- Usar alias para tablas
SELECT e.nombre, e.apellido, e.salario
FROM empleados e;
```

#### 5.2. Filtrado con WHERE

La cláusula `WHERE` filtra filas que cumplen una condición:

```sql
-- Empleados con salario mayor a 40000
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > 40000;

-- Empleados de un departamento específico
SELECT nombre, apellido, departamento_id
FROM empleados
WHERE departamento_id = 20;

-- Empleados activos
SELECT nombre, apellido
FROM empleados
WHERE activo = TRUE;
```

#### 5.3. Operadores de comparación

| Operador | Significado | Ejemplo |
| :-: | :-: | :-: |
| `=` | Igual a | `salario = 50000` |
| `<>` o `!=` | Distinto de | `departamento_id <> 10` |
| `>` | Mayor que | `salario > 40000` |
| `<` | Menor que | `salario < 30000` |
| `>=` | Mayor o igual que | `salario >= 35000` |
| `<=` | Menor o igual que | `salario <= 45000` |

#### 5.4. Operadores lógicos

| Operador | Significado | Ejemplo |
| :-: | :-: | :-: |
| `AND` | Conjunción (Y) | `salario > 40000 AND activo = TRUE` |
| `OR` | Disyunción (O) | `departamento_id = 10 OR departamento_id = 20` |
| `NOT` | Negación | `NOT activo = FALSE` |

Ejemplos:

```sql
-- Combinar con AND
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > 40000 AND activo = TRUE;

-- Combinar con OR
SELECT nombre, apellido, departamento_id
FROM empleados
WHERE departamento_id = 10 OR departamento_id = 20;

-- Usar paréntesis para precedencia
SELECT nombre, apellido, salario
FROM empleados
WHERE (salario > 40000 AND departamento_id = 20)
   OR (salario > 50000 AND activo = TRUE);
```

#### 5.5. Operadores especiales

```sql
-- BETWEEN: rango inclusivo
SELECT nombre, salario
FROM empleados
WHERE salario BETWEEN 35000 AND 45000;

-- IN: lista de valores
SELECT nombre, departamento_id
FROM empleados
WHERE departamento_id IN (10, 20, 30);

-- LIKE: coincidencia de patrones
SELECT nombre, apellido
FROM empleados
WHERE nombre LIKE 'J%';     -- Empieza con J

SELECT nombre, apellido
FROM empleados
WHERE apellido LIKE '%ez';  -- Termina con ez

SELECT nombre
FROM empleados
WHERE nombre LIKE '_a%';    -- Segunda letra es 'a'

-- IS NULL: verificar valores nulos
SELECT nombre, fecha_nacimiento
FROM empleados
WHERE fecha_nacimiento IS NULL;

-- IS NOT NULL: verificar valores no nulos
SELECT nombre, fecha_nacimiento
FROM empleados
WHERE fecha_nacimiento IS NOT NULL;
```

**Nota importante:** la comparación `= NULL` nunca funciona en SQL porque `NULL` representa un valor desconocido. Siempre use `IS NULL` o `IS NOT NULL`.

#### 5.6. Ordenamiento y limitación

```sql
-- Ordenar por salario descendente
SELECT nombre, apellido, salario
FROM empleados
ORDER BY salario DESC;

-- Ordenar por múltiples columnas
SELECT nombre, apellido, departamento_id, salario
FROM empleados
ORDER BY departamento_id ASC, salario DESC;

-- Limitar resultados
SELECT nombre, apellido, salario
FROM empleados
ORDER BY salario DESC
LIMIT 10;

-- Sintaxis estándar SQL:2008
SELECT nombre, apellido, salario
FROM empleados
ORDER BY salario DESC
FETCH FIRST 10 ROWS ONLY;
```

---

#### 6. Tres formas de contar filas

El conteo es una operación común con matices importantes.

#### 6.1. COUNT(*): contar todas las filas

Cuenta todas las filas independientemente de los valores, incluyendo `NULL`:

```sql
-- Total de empleados
SELECT COUNT(*) AS total_empleados
FROM empleados;

-- Top 5 departamentos con más empleados
SELECT departamento_id, COUNT(*) AS total
FROM empleados
GROUP BY departamento_id
ORDER BY COUNT(*) DESC
LIMIT 5;
```

#### 6.2. COUNT(DISTINCT columna): contar valores únicos

Útil cuando una columna tiene duplicados:

```sql
-- ¿Cuántos departamentos distintos tienen empleados?
SELECT COUNT(DISTINCT departamento_id) AS departamentos_con_empleados
FROM empleados;

-- ¿Cuántos apellidos distintos hay?
SELECT COUNT(DISTINCT apellido) AS apellidos_unicos
FROM empleados;
```

#### 6.3. Conteo condicional con SUM(CASE WHEN...)

Para contar filas que cumplen una condición específica:

```sql
-- Contar empleados activos e inactivos por departamento
SELECT
    departamento_id,
    SUM(CASE WHEN activo = TRUE THEN 1 ELSE 0 END) AS num_activos,
    SUM(CASE WHEN activo = FALSE THEN 1 ELSE 0 END) AS num_inactivos,
    SUM(CASE WHEN salario > 40000 THEN 1 ELSE 0 END) AS num_salario_alto
FROM empleados
GROUP BY departamento_id;
```

La expresión `CASE WHEN condicion THEN 1 ELSE 0 END` produce `1` para filas que cumplen la condición y `0` para las demás. Al sumar, se obtiene el conteo.

---

#### 7. Preguntas fundamentales sobre los datos

Antes de escribir consultas complejas, formule preguntas críticas sobre la calidad de los datos:

#### 7.1. ¿Esta columna tiene valores NULL, 0 o vacíos?

```sql
-- Detectar NULL
SELECT COUNT(*) AS nulos
FROM empleados
WHERE fecha_nacimiento IS NULL;

-- Detectar ceros
SELECT COUNT(*) AS ceros
FROM empleados
WHERE salario = 0;

-- Detectar cadenas vacías
SELECT COUNT(*) AS vacios
FROM empleados
WHERE nombre = '';
```

#### 7.2. ¿Cuántos valores distintos tiene esta columna?

```sql
SELECT COUNT(DISTINCT departamento_id) AS departamentos_distintos
FROM empleados;

SELECT DISTINCT departamento_id
FROM empleados
ORDER BY departamento_id;
```

#### 7.3. ¿Hay valores duplicados?

```sql
-- Detectar duplicados
SELECT departamento_id, COUNT(*) AS frecuencia
FROM empleados
GROUP BY departamento_id
HAVING COUNT(*) > 1;
```

#### 7.4. ¿La columna id de tabla A siempre coincide en tabla B?

```sql
-- Detectar empleados sin departamento correspondiente
SELECT e.id_empleado, e.departamento_id
FROM empleados e
LEFT JOIN departamentos d ON e.departamento_id = d.id_departamento
WHERE d.id_departamento IS NULL;
```

---

#### 8. Funciones de agregación y agrupamiento

#### 8.1. Funciones de agregación

Realizan cálculos sobre conjuntos de valores y devuelven un único valor:

| Función | Significado | Ejemplo |
| :-: | :-: | :-: |
| `COUNT(*)` | Contar filas | `COUNT(*)` |
| `COUNT(columna)` | Contar valores no nulos | `COUNT(salario)` |
| `SUM(columna)` | Sumar valores | `SUM(salario)` |
| `AVG(columna)` | Promedio | `AVG(salario)` |
| `MIN(columna)` | Valor mínimo | `MIN(salario)` |
| `MAX(columna)` | Valor máximo | `MAX(salario)` |

Ejemplo:

```sql
SELECT COUNT(*) AS total,
       AVG(salario) AS promedio,
       MIN(salario) AS minimo,
       MAX(salario) AS maximo,
       SUM(salario) AS nomina_total
FROM empleados;
```

#### 8.2. GROUP BY

Agrupa filas con los mismos valores en columnas especificadas:

```sql
-- Empleados por departamento
SELECT departamento_id, COUNT(*) AS total_empleados
FROM empleados
GROUP BY departamento_id;

-- Estadísticas por departamento
SELECT departamento_id,
       COUNT(*) AS total_empleados,
       AVG(salario) AS salario_promedio,
       SUM(salario) AS nomina_departamento
FROM empleados
GROUP BY departamento_id;
```

#### 8.3. HAVING

Filtra grupos después de la agregación (a diferencia de `WHERE` que filtra filas individuales):

```sql
-- Departamentos con más de 2 empleados activos
SELECT departamento_id, COUNT(*) AS total_empleados
FROM empleados
WHERE activo = TRUE
GROUP BY departamento_id
HAVING COUNT(*) > 2;

-- Departamentos con salario promedio mayor a 40000
SELECT departamento_id, AVG(salario) AS salario_promedio
FROM empleados
GROUP BY departamento_id
HAVING AVG(salario) > 40000;
```

**Encontrar duplicados con HAVING:**

```sql
-- Encontrar correos duplicados
SELECT correo, COUNT(*) AS frecuencia,
       STRING_AGG(nombre, ', ') AS nombres
FROM clientes
GROUP BY correo
HAVING COUNT(*) > 1;
```

---

#### 9. Funciones en PostgreSQL

#### 9.1. Funciones numéricas

```sql
SELECT 
    ABS(-5) AS valor_absoluto,           -- 5
    ROUND(3.14159, 2) AS redondeado,     -- 3.14
    CEIL(3.2) AS techo,                  -- 4
    FLOOR(3.8) AS piso,                  -- 3
    MOD(10, 3) AS modulo,                -- 1
    POWER(2, 3) AS potencia,             -- 8
    SQRT(16) AS raiz;                    -- 4
```

#### 9.2. Funciones de cadena

```sql
SELECT 
    LENGTH('Hola') AS longitud,          -- 4
    UPPER('hola') AS mayusculas,         -- 'HOLA'
    LOWER('HOLA') AS minusculas,         -- 'hola'
    TRIM('  Hola  ') AS sin_espacios,    -- 'Hola'
    SUBSTRING('Hola Mundo', 1, 4) AS subcadena,  -- 'Hola'
    CONCAT('Hola', ' ', 'Mundo') AS concatenado,  -- 'Hola Mundo'
    REPLACE('Hola', 'a', 'o') AS reemplazo;       -- 'Holo'
```

#### 9.3. Funciones de fecha y hora

```sql
-- Fecha y hora actual
SELECT 
    CURRENT_DATE AS fecha_hoy,
    CURRENT_TIME AS hora_actual,
    CURRENT_TIMESTAMP AS fecha_hora_actual;

-- Extraer componentes
SELECT 
    EXTRACT(YEAR FROM fecha_nacimiento) AS anio,
    EXTRACT(MONTH FROM fecha_nacimiento) AS mes,
    EXTRACT(DAY FROM fecha_nacimiento) AS dia
FROM empleados;

-- Calcular edad
SELECT 
    nombre,
    AGE(CURRENT_DATE, fecha_nacimiento) AS edad_intervalo,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, fecha_nacimiento)) AS edad_anios
FROM empleados;

-- Manipular fechas
SELECT 
    fecha_contratacion,
    fecha_contratacion + INTERVAL '30 days' AS mas_30_dias,
    fecha_contratacion - INTERVAL '1 year' AS menos_1_anio
FROM empleados;
```

#### 9.4. Funciones de conversión

```sql
-- CAST: conversión estándar
SELECT 
    CAST(salario AS INTEGER) AS salario_entero,
    CAST(fecha_contratacion AS TEXT) AS fecha_texto
FROM empleados;

-- Sintaxis alternativa en PostgreSQL
SELECT 
    salario::INTEGER AS salario_entero,
    fecha_contratacion::TEXT AS fecha_texto
FROM empleados;

-- COALESCE: primer valor no nulo
SELECT 
    nombre,
    COALESCE(fecha_nacimiento, DATE '1900-01-01') AS fecha
FROM empleados;

-- NULLIF: NULL si dos valores son iguales
SELECT 
    NULLIF(departamento_id, 0) AS departamento
FROM empleados;
```

---

#### 10. Joins: combinando tablas

#### 10.1. Reglas prácticas para JOINs

Los JOINs pueden volverse complejos. Tres reglas para el 90% de los casos:

**Regla 1:** Use únicamente `LEFT JOIN` e `INNER JOIN`. Aunque existen `RIGHT JOIN`, `FULL OUTER JOIN` y `CROSS JOIN`, los dos primeros son suficientes para la mayoría de necesidades.

**Regla 2:** Incluya una sola condición en el `ON`:

```sql
tabla1 LEFT JOIN tabla2 ON tabla1.columna = tabla2.otra_columna
```

**Regla 3:** Una de las columnas unidas debe ser única. Si ninguna es única, se obtienen resultados inesperados por producto cartesiano parcial.

#### 10.2. INNER JOIN

Devuelve solo filas con coincidencias en ambas tablas:

```sql
-- Empleados con su departamento
SELECT e.nombre, e.apellido, d.nombre_departamento
FROM empleados e
INNER JOIN departamentos d ON e.departamento_id = d.id_departamento;
```

#### 10.3. LEFT JOIN paso a paso

Devuelve todas las filas de la tabla izquierda y las coincidencias de la derecha:

```sql
SELECT e.nombre, d.nombre_departamento
FROM empleados e
LEFT JOIN departamentos d ON e.departamento_id = d.id_departamento;
```

**Proceso lógico:**

**Paso 1:** Combinar cada fila de `empleados` con cada fila de `departamentos` (producto cartesiano temporal).

**Paso 2:** Conservar solo filas donde `e.departamento_id = d.id_departamento`.

**Paso 3:** Reincorporar filas de `empleados` sin coincidencia, rellenando con `NULL` las columnas de `departamentos`.

Este último paso distingue `LEFT JOIN` de `INNER JOIN` y es útil para identificar filas huérfanas:

```sql
-- Empleados sin departamento asignado
SELECT e.nombre, e.apellido
FROM empleados e
LEFT JOIN departamentos d ON e.departamento_id = d.id_departamento
WHERE d.id_departamento IS NULL;
```

---

#### 11. Subconsultas

Las *subconsultas* son consultas anidadas dentro de otra consulta principal.

#### 11.1. Subconsultas escalares

Devuelven un único valor:

```sql
-- Empleados que ganan más que el promedio
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario)
    FROM empleados
);
```

#### 11.2. Subconsultas de columna

Devuelven una columna con múltiples filas:

```sql
-- Empleados de departamentos en Edificio A
SELECT nombre, apellido
FROM empleados
WHERE departamento_id IN (
    SELECT id_departamento
    FROM departamentos
    WHERE ubicacion LIKE '%Edificio A%'
);

-- Empleados con salario mayor que todos los del departamento 10
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > ALL (
    SELECT salario
    FROM empleados
    WHERE departamento_id = 10
);
```

#### 11.3. Subconsultas de tabla

Devuelven múltiples columnas y filas, usadas en `FROM`:

```sql
-- Departamentos con salario promedio superior a 40000
SELECT departamento, salario_promedio
FROM (
    SELECT 
        d.nombre_departamento AS departamento,
        AVG(e.salario) AS salario_promedio
    FROM empleados e
    INNER JOIN departamentos d ON e.departamento_id = d.id_departamento
    GROUP BY d.id_departamento, d.nombre_departamento
) AS promedios
WHERE salario_promedio > 40000;
```

#### 11.4. Subconsultas correlacionadas

Referencian columnas de la consulta externa y se ejecutan una vez por cada fila:

```sql
-- Empleados que ganan más que el promedio de su departamento
SELECT e1.nombre, e1.apellido, e1.salario, e1.departamento_id
FROM empleados e1
WHERE e1.salario > (
    SELECT AVG(e2.salario)
    FROM empleados e2
    WHERE e2.departamento_id = e1.departamento_id
);
```

#### 11.5. EXISTS y NOT EXISTS

Verifican la existencia de filas:

```sql
-- Departamentos que tienen al menos un empleado
SELECT nombre_departamento
FROM departamentos
WHERE EXISTS (
    SELECT 1
    FROM empleados
    WHERE empleados.departamento_id = departamentos.id_departamento
);

-- Departamentos sin empleados
SELECT nombre_departamento
FROM departamentos
WHERE NOT EXISTS (
    SELECT 1
    FROM empleados
    WHERE empleados.departamento_id = departamentos.id_departamento
);
```

---

#### 12. Funciones de ventana

Las *funciones de ventana* permiten referenciar valores de otras filas sin colapsar el resultado, a diferencia de las funciones de agregación.

#### 12.1. Sintaxis básica

```sql
funcion() OVER (
    [PARTITION BY <columnas>]
    [ORDER BY <columnas>]
    [window_frame]
)
```

- `PARTITION BY`: divide datos en particiones independientes (similar a `GROUP BY` pero sin colapsar)
- `ORDER BY`: define el orden dentro de cada partición

#### 12.2. Ejemplo: días entre eventos

Considere una tabla `clima` con eventos meteorológicos:

```sql
CREATE TABLE clima (
    tipo VARCHAR(20),
    dia INTEGER
);

INSERT INTO clima (tipo, dia) VALUES
('rain', 9), ('thunderstorm', 11), ('rain', 13),
('rain', 21), ('thunderstorm', 22), ('rain', 30),
('thunderstorm', 36), ('rain', 38), ('thunderstorm', 41),
('rain', 48);
```

Calcular días transcurridos desde el evento anterior del mismo tipo:

```sql
SELECT tipo, dia,
       dia - LAG(dia) OVER (PARTITION BY tipo ORDER BY dia ASC) AS dias_desde_anterior
FROM clima
ORDER BY dia ASC;
```

Resultado:

```
┌──────────────┬─────┬─────────────────────┐
│ tipo         │ dia │ dias_desde_anterior │
├──────────────┼─────┼─────────────────────┤
│ rain         │   9 │ NULL                │
│ thunderstorm │  11 │ NULL                │
│ rain         │  13 │ 4                   │
│ rain         │  21 │ 8                   │
│ thunderstorm │  22 │ 11                  │
│ rain         │  30 │ 9                   │
│ thunderstorm │  36 │ 14                  │
│ rain         │  38 │ 8                   │
│ thunderstorm │  41 │ 5                   │
│ rain         │  48 │ 10                  │
└──────────────┴─────┴─────────────────────┘
```

`LAG(dia)` devuelve el valor de `dia` en la fila anterior dentro de la misma partición. Las primeras filas de cada partición producen `NULL`.

#### 12.3. Otras funciones de ventana comunes

| Función | Significado |
| :-: | :-- |
| `ROW_NUMBER()` | Número de fila dentro de la partición |
| `RANK()` | Rango con saltos en empates |
| `DENSE_RANK()` | Rango sin saltos en empates |
| `LAG(columna, n)` | Valor de n filas antes |
| `LEAD(columna, n)` | Valor de n filas después |
| `SUM(columna) OVER (...)` | Suma acumulada |
| `AVG(columna) OVER (...)` | Promedio móvil |

Ejemplo con `ROW_NUMBER`:

```sql
-- Numerar empleados por salario dentro de cada departamento
SELECT 
    departamento_id,
    nombre,
    salario,
    ROW_NUMBER() OVER (PARTITION BY departamento_id ORDER BY salario DESC) AS ranking
FROM empleados;
```

---

#### 13. Operaciones de conjuntos

SQL permite combinar resultados de múltiples consultas:

```sql
-- UNION: combina eliminando duplicados
SELECT nombre, apellido FROM empleados_antiguos
UNION
SELECT nombre, apellido FROM empleados_nuevos;

-- UNION ALL: combina incluyendo duplicados
SELECT nombre, apellido FROM empleados_antiguos
UNION ALL
SELECT nombre, apellido FROM empleados_nuevos;

-- INTERSECT: solo filas comunes
SELECT departamento_id FROM empleados WHERE salario > 50000
INTERSECT
SELECT departamento_id FROM empleados WHERE activo = TRUE;

-- EXCEPT: filas de primera consulta que no están en la segunda
SELECT departamento_id FROM departamentos
EXCEPT
SELECT DISTINCT departamento_id FROM empleados;
```

---

#### 14. Vistas

Las *vistas* son consultas almacenadas que se comportan como tablas virtuales:

```sql
-- Crear una vista
CREATE VIEW vista_empleados_activos AS
SELECT id_empleado, nombre, apellido, salario, departamento_id
FROM empleados
WHERE activo = TRUE;

-- Consultar una vista
SELECT * FROM vista_empleados_activos;

-- Vista con agregación
CREATE VIEW vista_resumen_departamentos AS
SELECT departamento_id,
       COUNT(*) AS total_empleados,
       AVG(salario) AS salario_promedio,
       SUM(salario) AS nomina_total
FROM empleados
GROUP BY departamento_id;

-- Eliminar una vista
DROP VIEW vista_empleados_activos;
```

---

#### 15. Inserción, actualización y eliminación de datos

#### 15.1. INSERT

```sql
-- Insertar una fila
INSERT INTO empleados (id_empleado, nombre, apellido, salario, departamento_id)
VALUES (11, 'Roberto', 'Díaz', 39000.00, 10);

-- Insertar múltiples filas
INSERT INTO departamentos (id_departamento, nombre_departamento, ubicacion, presupuesto)
VALUES 
    (50, 'Investigación', 'Edificio D, Piso 1', 900000.00),
    (60, 'Legal', 'Edificio A, Piso 3', 700000.00);

-- Insertar desde consulta
INSERT INTO empleados_archivados (id_empleado, nombre, apellido, fecha_baja)
SELECT id_empleado, nombre, apellido, CURRENT_DATE
FROM empleados
WHERE activo = FALSE;
```

#### 15.2. UPDATE

```sql
-- Actualizar una fila
UPDATE empleados
SET salario = 45000.00
WHERE id_empleado = 1;

-- Actualizar múltiples columnas
UPDATE empleados
SET salario = 48000.00, departamento_id = 20
WHERE id_empleado = 2;

-- Actualizar con cálculo
UPDATE empleados
SET salario = salario * 1.10
WHERE departamento_id = 10;
```

**Importante:** siempre use `WHERE` en `UPDATE` para evitar actualizar todas las filas.

#### 15.3. DELETE

```sql
-- Eliminar una fila
DELETE FROM empleados
WHERE id_empleado = 10;

-- Eliminar empleados inactivos
DELETE FROM empleados
WHERE activo = FALSE;
```

**Importante:** siempre use `WHERE` en `DELETE` para evitar eliminar todas las filas.

#### 15.4. TRUNCATE

Elimina todas las filas más eficientemente que `DELETE`:

```sql
TRUNCATE TABLE empleados;
```

---

#### 16. Funciones, procedimientos y disparadores

#### 16.1. Funciones definidas por el usuario

Las funciones aceptan parámetros y devuelven un valor o tabla:

**Función escalar:**

```sql
-- Función que calcula edad
CREATE OR REPLACE FUNCTION calcular_edad(fecha_nac DATE)
RETURNS INTEGER
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, fecha_nac));
END;
$$ LANGUAGE plpgsql;

-- Uso
SELECT nombre, calcular_edad(fecha_nacimiento) AS edad
FROM empleados;
```

**Función que retorna tabla:**

```sql
-- Función que retorna empleados de un departamento
CREATE OR REPLACE FUNCTION empleados_por_depto(id_depto INTEGER)
RETURNS TABLE (
    id_empleado INTEGER,
    nombre VARCHAR(50),
    apellido VARCHAR(50),
    salario NUMERIC
)
AS $$
BEGIN
    RETURN QUERY
    SELECT e.id_empleado, e.nombre, e.apellido, e.salario
    FROM empleados e
    WHERE e.departamento_id = id_depto;
END;
$$ LANGUAGE plpgsql;

-- Uso
SELECT * FROM empleados_por_depto(20);
```

#### 16.2. Procedimientos almacenados

Los procedimientos pueden contener lógica compleja y operaciones de modificación, pero no devuelven valores directamente:

```sql
-- Procedimiento para aplicar aumento salarial
CREATE OR REPLACE PROCEDURE aplicar_aumento_salarial(
    id_depto INTEGER,
    porcentaje NUMERIC
)
AS $$
BEGIN
    UPDATE empleados
    SET salario = salario * (1 + porcentaje / 100.0)
    WHERE departamento_id = id_depto;
    
    INSERT INTO historial_aumentos (departamento_id, porcentaje, fecha)
    VALUES (id_depto, porcentaje, CURRENT_DATE);
    
    COMMIT;
END;
$$ LANGUAGE plpgsql;

-- Ejecución
CALL aplicar_aumento_salarial(10, 5.0);
```

#### 16.3. Disparadores (Triggers)

Los triggers se ejecutan automáticamente ante eventos específicos (INSERT, UPDATE, DELETE):

**Ejemplo: auditoría de cambios de salario**

```sql
-- Tabla de auditoría
CREATE TABLE empleados_auditoria (
    id_auditoria SERIAL PRIMARY KEY,
    id_empleado INTEGER,
    accion VARCHAR(10),
    salario_anterior NUMERIC,
    salario_nuevo NUMERIC,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario VARCHAR(50)
);

-- Función trigger
CREATE OR REPLACE FUNCTION auditar_cambio_salario()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' AND OLD.salario IS DISTINCT FROM NEW.salario THEN
        INSERT INTO empleados_auditoria (
            id_empleado, accion, salario_anterior, salario_nuevo, usuario
        )
        VALUES (
            NEW.id_empleado, 'UPDATE', OLD.salario, NEW.salario, CURRENT_USER
        );
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear el trigger
CREATE TRIGGER trigger_auditoria_salario
AFTER UPDATE ON empleados
FOR EACH ROW
EXECUTE FUNCTION auditar_cambio_salario();
```

Ahora, cada vez que se actualice el salario de un empleado, se registrará automáticamente en la tabla de auditoría.

---

#### 17. Transacciones

Las transacciones agrupan múltiples operaciones en una unidad atómica que cumple propiedades ACID (Atomicity, Consistency, Isolation, Durability):

```sql
-- Iniciar transacción
BEGIN;

-- Realizar operaciones
UPDATE empleados SET salario = 45000 WHERE id_empleado = 1;
UPDATE empleados SET salario = 48000 WHERE id_empleado = 2;

-- Confirmar transacción
COMMIT;

-- O cancelar (revertir todos los cambios)
-- ROLLBACK;
```

Ejemplo completo:

```sql
BEGIN;

-- Transferir empleado
UPDATE empleados
SET departamento_id = 20
WHERE id_empleado = 5;

-- Registrar cambio
INSERT INTO historial_cambios (id_empleado, cambio, fecha)
VALUES (5, 'Cambio de departamento', CURRENT_DATE);

-- Si todo es correcto
COMMIT;

-- Si hay error:
-- ROLLBACK;
```

---

#### 18. Administración elemental de bases de datos en PostgreSQL

#### 18.1. Gestión de roles y usuarios

Los *roles* son entidades que pueden tener permisos. En PostgreSQL, usuarios y roles son conceptos similares.

**Creación de roles:**

```sql
-- Rol sin privilegios de login (grupo)
CREATE ROLE analista;

-- Rol con login (usuario)
CREATE ROLE juan_perez WITH LOGIN PASSWORD 'contraseña_segura';

-- Rol con múltiples opciones
CREATE ROLE administrador_db WITH
    LOGIN
    PASSWORD 'contraseña_forte'
    CREATEDB
    CREATEROLE
    VALID UNTIL '2027-12-31';

-- Ver roles existentes
SELECT rolname, rolsuper, rolcreaterole, rolcreatedb, rolcanlogin
FROM pg_roles
WHERE rolname NOT LIKE 'pg_%'
ORDER BY rolname;
```

**Atributos de roles:**

| Atributo | Descripción |
| :-: | :-- |
| `SUPERUSER` | Todos los privilegios (equivalente a root) |
| `CREATEDB` | Puede crear bases de datos |
| `CREATEROLE` | Puede crear/modificar otros roles |
| `LOGIN` | Puede conectarse |
| `INHERIT` | Hereda privilegios de roles miembro |

#### 18.2. Asignación de permisos (GRANT)

```sql
-- Todos los privilegios sobre una tabla
GRANT ALL PRIVILEGES ON TABLE empleados TO juan_perez;

-- Privilegios específicos
GRANT SELECT, INSERT ON TABLE empleados TO analista;

-- Solo lectura
GRANT SELECT ON TABLE empleados, departamentos TO rol_consulta;

-- Privilegios a nivel de columna
GRANT SELECT (id_empleado, nombre, apellido) ON TABLE empleados TO rol_publico;

-- Uso de esquema
GRANT USAGE ON SCHEMA public TO analista;

-- Ejecución de función
GRANT EXECUTE ON FUNCTION calcular_edad(DATE) TO public;

-- Membresía en rol
GRANT analista TO juan_perez;
```

#### 18.3. Revocación de permisos (REVOKE)

```sql
-- Revocar todos los privilegios
REVOKE ALL PRIVILEGES ON TABLE empleados FROM juan_perez;

-- Revocar privilegios específicos
REVOKE INSERT, UPDATE ON TABLE empleados FROM analista;

-- Revocar membresía
REVOKE analista FROM juan_perez;

-- Revocar con CASCADE (elimina permisos derivados)
REVOKE GRANT OPTION FOR SELECT ON TABLE empleados FROM analista CASCADE;
```

#### 18.4. Respaldo de la base de datos

El respaldo es crucial para recuperación ante desastres. PostgreSQL proporciona la herramienta `pg_dump`.

#### 18.4.1. Respaldo del esquema (estructura)

El esquema incluye tablas, vistas, funciones, triggers, índices, constraints:

```bash
# Solo esquema (sin datos)
pg_dump -s empresa_db > esquema_empresa.sql

# Esquema sin propietarios ni privilegios (más portable)
pg_dump -s --no-owner --no-privileges empresa_db > esquema_limpio.sql

# Solo tablas específicas
pg_dump -s -t empleados -t departamentos empresa_db > esquema_rrhh.sql
```

Contenido típico del archivo de esquema:

```sql
-- Tablas
CREATE TABLE departamentos (
    id_departamento INTEGER PRIMARY KEY,
    nombre_departamento VARCHAR(100) NOT NULL UNIQUE,
    ubicacion VARCHAR(200),
    presupuesto NUMERIC(12,2)
);

CREATE TABLE empleados (
    id_empleado INTEGER PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    salario NUMERIC(10,2) CHECK (salario > 0),
    departamento_id INTEGER,
    CONSTRAINT fk_empleado_departamento
        FOREIGN KEY (departamento_id)
        REFERENCES departamentos(id_departamento)
);

-- Índices
CREATE INDEX idx_empleado_departamento ON empleados(departamento_id);

-- Funciones
CREATE FUNCTION calcular_edad(fecha_nac DATE)
RETURNS INTEGER AS $$ ... $$ LANGUAGE plpgsql;

-- Vistas
CREATE VIEW vista_empleados_activos AS ...;

-- Triggers
CREATE TRIGGER trigger_auditoria_salario ...;
```

#### 18.4.2. Respaldo de los datos

Solo la información almacenada, sin estructura:

```bash
# Solo datos (sin esquema)
pg_dump -a empresa_db > datos_empresa.sql

# Datos en formato INSERT (más portable)
pg_dump -a --inserts empresa_db > datos_insert.sql

# Solo tablas específicas
pg_dump -a -t empleados -t departamentos empresa_db > datos_rrhh.sql
```

Contenido típico del archivo de datos:

```sql
COPY departamentos (id_departamento, nombre_departamento, ubicacion, presupuesto) FROM stdin;
10	Recursos Humanos	Edificio A Piso 2	500000.00
20	Tecnología	Edificio B Piso 3	1200000.00
30	Ventas	Edificio A Piso 1	800000.00
\.

COPY empleados (id_empleado, nombre, apellido, salario, departamento_id) FROM stdin;
1	Juan	Pérez	35000.00	10
2	María	González	42000.00	20
\.
```

#### 18.4.3. Respaldo completo

Incluye esquema y datos:

```bash
# Respaldo completo
pg_dump empresa_db > respaldo_completo.sql

# Con compresión
pg_dump empresa_db | gzip > respaldo_completo.sql.gz

# Formato personalizado (más eficiente para respaldos grandes)
pg_dump -Fc empresa_db > respaldo_completo.dump
```

El formato personalizado (`-Fc`) es más eficiente para bases de datos grandes porque permite compresión y restauración selectiva.

#### 18.5. Restauración de respaldos

**Restaurar esquema:**

```bash
psql empresa_db < esquema_empresa.sql
```

**Restaurar datos:**

```bash
psql empresa_db < datos_empresa.sql
```

**Restaurar respaldo completo:**

```bash
psql empresa_db < respaldo_completo.sql

# O para formato personalizado
pg_restore -d empresa_db respaldo_completo.dump
```

**Consideraciones importantes:**

- **Orden de restauración:** Primero el esquema, luego los datos
- **Integridad referencial:** Las tablas deben crearse en orden correcto (padres antes que hijos)
- **Secuencias:** Restablecer valores de secuencias después de cargar datos
- **Índices:** Crear índices después de cargar datos para mejor rendimiento
- **Validación:** Verificar integridad de datos después de restauración

#### 18.6. Buenas prácticas de administración

**Seguridad:**

- Utilizar contraseñas fuertes y rotarlas periódicamente
- Aplicar principio de mínimo privilegio
- Auditar accesos y cambios regularmente
- Cifrar conexiones (SSL/TLS)
- No usar cuentas de superusuario para aplicaciones

**Rendimiento:**

- Mantener estadísticas actualizadas con `ANALYZE`
- Reconstruir índices periódicamente con `REINDEX`
- Monitorear consultas lentas
- Implementar particionamiento para tablas grandes

**Mantenimiento:**

- Realizar respaldos automáticos y programados
- Verificar integridad de respaldos periódicamente
- Documentar cambios en el esquema
- Mantener actualizado el sistema gestor de base de datos

---

#### 19. Resumen de la anatomía de SELECT

Para consolidar lo expuesto, la siguiente tabla resume la anatomía completa de una instrucción `SELECT` con su orden de escritura y su orden de ejecución:

| Orden de escritura | Orden de ejecución | Cláusula | Propósito |
| :-: | :-: | :-- | :-- |
| 1 | 5 | `SELECT` | Proyecta las columnas finales |
| 2 | 1 | `FROM` | Identifica la tabla origen y realiza JOINs |
| 3 | 2 | `WHERE` | Filtra filas individuales |
| 4 | 3 | `GROUP BY` | Agrupa filas por columnas |
| 5 | 4 | `HAVING` | Filtra grupos |
| 6 | 6 | `ORDER BY` | Ordena el resultado final |
| 7 | 7 | `LIMIT` / `FETCH` | Limita el número de filas |

Esta distinción entre el orden de escritura y el orden de ejecución es fundamental para comprender por qué ciertas construcciones son válidas y otras no. Por ejemplo, un alias definido en `SELECT` no puede utilizarse en `WHERE` (porque `WHERE` se ejecuta antes), pero sí puede utilizarse en `ORDER BY` (porque `ORDER BY` se ejecuta después).

---

#### Referencias

1. Date, C. J. (2004). *An Introduction to Database Systems* (8th ed.). Pearson.
2. Elmasri, R., & Navathe, S. (2016). *Fundamentals of Database Systems* (7th ed.). Pearson.
3. Evans, J. (2024). *SQL* [Zine]. https://jvns.ca/
4. ISO/IEC 9075:2023. *Information technology — Database languages — SQL*. International Organization for Standardization.
5. PostgreSQL Global Development Group. PostgreSQL Documentation. https://www.postgresql.org/docs/
6. Silberschatz, A., Korth, H. F., & Sudarshan, S. (2020). *Database System Concepts* (7th ed.). McGraw-Hill.
