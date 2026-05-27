### Lectura 04: Fundamentos de Bases de Datos Relacionales y SQL: Teoría y Práctica

**Dr. Jesús Zavala Ruiz**  

Mayo 2026

---

#### Resumen

El presente documento constituye una lectura académica integral diseñada para estudiantes de licenciatura que inician su formación en gestión y diseño de bases de datos relacionales. Su propósito fundamental es establecer un puente riguroso entre los fundamentos teóricos del modelo relacional y la implementación práctica mediante el Lenguaje de Consulta Estructurado (SQL), siguiendo una secuencia operativa que prioriza la creación del esquema de base de datos, la definición ordenada de tablas para preservar la integridad referencial, y el poblamiento controlado de datos. Se integran definiciones canónicas de Date (2019) y Mata-Toledo (2000), citadas conforme a APA 7ª edición, y se articula cada concepto con la arquitectura modular de SQL. Como eje pedagógico, se emplea la base de datos Pagila, analizando la práctica profesional de separar el respaldo del esquema y el de los datos, sus ventajas operativas y sus riesgos de desincronización. Finalmente, se enmarca la gestión de respaldos como política de seguridad institucional que exige periodicidad, automatización y verificación de restauración, con énfasis en las mejores prácticas para entornos de producción.

**Palabras clave:** modelo relacional, SQL, integridad referencial, esquema de base de datos, Pagila, PostgreSQL, respaldo de datos, política de seguridad, producción.

#### 1. Introducción

La gestión de datos estructurados constituye uno de los pilares fundamentales de la informática contemporánea. Desde los sistemas transaccionales bancarios hasta las plataformas de análisis de datos masivos, la organización, conservación y consulta de información reposa en gran medida sobre paradigmas relacionales que han demostrado una resiliencia excepcional a lo largo de más de cinco décadas. La propuesta original de Edgar F. Codd, publicada en 1970, introdujo un marco matemático que separaba la representación lógica de los detalles físicos de almacenamiento, garantizando independencia de datos, consistencia semántica y capacidad de razonamiento formal sobre la información almacenada. Sin embargo, la transición desde la teoría pura hacia la implementación operativa requiere un lenguaje estandarizado que permita a los profesionales interactuar con los sistemas de gestión de bases de datos (SGBD) de manera precisa, segura y eficiente. Dicho lenguaje es SQL, cuya especificación ISO/IEC 9075 ha evolucionado para integrar capacidades de definición, manipulación, control y administración transaccional.

El presente documento tiene como objetivo presentar de manera sistemática los fundamentos teóricos del modelo relacional y su correlato práctico en SQL, utilizando como estudio de caso la base de datos Pagila. Esta base de datos, derivada del esquema clásico "DVD Rental", ha sido ampliamente adoptada en entornos académicos debido a su riqueza estructural, su adherencia a principios de normalización y su utilidad para ilustrar patrones relacionales complejos como relaciones muchos a muchos, integridad referencial y jerarquías de datos. A lo largo de esta lectura, se integrarán definiciones canónicas extraídas de obras fundacionales como las de Date (2019) y Mata-Toledo (2000), citadas conforme a la séptima edición de las normas APA. Cada concepto teórico será contrastado con ejemplos ejecutables sobre PostgreSQL 18, permitiendo al estudiante verificar empíricamente la correspondencia entre el marco formal y la implementación técnica.

La estructura del documento sigue una progresión lógica que refleja la secuencia de implementación física del modelo relacional: inicialmente se examinan los constructores teóricos del modelo relacional y los principios de integridad; posteriormente, se analiza el proceso de transformación del diseño conceptual al esquema lógico; a continuación, se desglosa la arquitectura modular de SQL y su alineación con el álgebra relacional; el cuarto apartado establece la secuencia operativa de implementación física, priorizando la creación del esquema de base de datos, el orden de definición de tablas y el poblamiento de datos; el quinto apartado se dedica al análisis del esquema Pagila como caso de estudio, examinando la práctica de separar respaldos de esquema y datos; el sexto apartado presenta una discusión crítica sobre teoría versus práctica, normalización y seguridad; finalmente, se ofrecen conclusiones que sintetizan los hallazgos y proyectan líneas de estudio avanzado. Esta organización responde a la necesidad de formar profesionales capaces no solo de escribir consultas, sino de comprender las implicaciones estructurales, semánticas y operativas de cada decisión de diseño.

#### 2. Fundamentos Teóricos: Constructores del Modelo Relacional y Principios de Integridad

El modelo relacional se fundamenta en una estructura matemática precisa que permite representar la información mediante conjuntos ordenados y reglas de inferencia verificables. Contrario a la percepción simplista que equipara bases de datos relacionales con meras tablas bidimensionales, la teoría relacional exige una distinción rigurosa entre valores, variables y restricciones. Date (2019) establece que la confusión terminológica entre conceptos como relación, relvar, tupla y atributo ha generado malentendidos pedagógicos y técnicos que dificultan la comprensión profunda del paradigma. Por ello, resulta indispensable definir cada constructor con precisión formal antes de abordar su implementación en SQL.

#### 2.1 Constructores Fundamentales

Una **relación** se define como "un valor que consiste en un encabezado (conjunto de pares nombre-atributo/tipo) y un cuerpo (conjunto de tuplas que comparten ese encabezado)" (Date, 2019, p. 38). La relación es, por tanto, un estado inmutable en un instante dado, análogo a una proposición lógica que puede ser verdadera o falsa respecto al dominio modelado.

En contraste, un **relvar** (variable de relación) es "una variable de relación: una variable cuyo valor en cualquier momento es una relación. Los relvars son los objetos que los usuarios actualizan mediante operaciones de inserción, eliminación y modificación" (Date, 2019, p. 25). Esta distinción es crucial: mientras que una consulta `SELECT` produce una relación temporal, una tabla base en PostgreSQL representa un relvar cuya extensión varía con el tiempo. La noción de relvar permite comprender que las operaciones de actualización (`INSERT`, `UPDATE`, `DELETE`) no modifican valores, sino que asignan nuevos valores a variables, preservando la consistencia lógica del modelo.

Un **atributo** es "un par nombre-atributo/tipo, donde el nombre identifica la columna y el tipo especifica el conjunto de valores permitidos" (Date, 2019, p. 26). Cada atributo toma sus valores de exactamente un **dominio**, entendido como "un conjunto de valores atómicos del mismo tipo; cada atributo de una relación toma sus valores de exactamente un dominio" (Date, 2019, p. 38). La atomicidad es un principio cardinal: "un valor es atómico si y solo si no tiene componentes identificables desde el punto de vista del DBMS" (Date, 2019, p. 66). Esta restricción garantiza que cada celda en una relación contenga un único valor indivisible, eliminando estructuras repetitivas y facilitando la aplicación de operadores algebraicos.

Una **tupla** es "una estructura de datos que representa un hecho único sobre el dominio modelado; en una relación, todas las tuplas comparten el mismo encabezado" (Mata-Toledo & Cushman, 2000, p. 12). La unicidad de tuplas es una propiedad inherente al modelo: no pueden existir dos tuplas idénticas en una relación, lo que implica que cada tupla es distinguible por al menos un conjunto de atributos. Dicho conjunto constituye una **superclave**, definida como "un conjunto de atributos de un relvar R tal que, en cualquier valor posible de R, no existen dos tuplas distintas que coincidan en todos los atributos de ese conjunto" (Date, 2019, p. 10). Una superclave mínima, es decir, aquella de la cual ninguna parte propia es también superclave, se denomina **clave candidata**: "una clave candidata es una superclave mínima: ninguna de sus partes propias es también una superclave" (Mata-Toledo & Cushman, 2000, p. 41). De entre las claves candidatas, el diseñador selecciona una como **clave primaria**, "la clave candidata seleccionada por el diseñador para identificar de manera preferente las tuplas de un relvar" (Mata-Toledo & Cushman, 2000, p. 42).

La **clave foránea** es "un conjunto de atributos de un relvar R2 cuyos valores deben coincidir con los valores de una clave candidata de otro relvar R1 (o del mismo R1), o bien ser nulos si está permitido" (Date, 2019, p. 12). Este constructo materializa la **integridad referencial**, una restricción que garantiza que las asociaciones entre relvars sean semánticamente consistentes. La **integridad de entidad**, por su parte, establece que ningún atributo de la clave primaria puede tener valor nulo, asegurando que cada tupla sea identificable de manera unívoca. Estas restricciones no son meras validaciones técnicas; son la formalización de reglas de negocio en la estructura de datos, y su omisión puede generar inconsistencias que comprometan la confiabilidad del sistema.

#### 2.2 Unicidad de Atributos y Valores Nulos

La **condición de unicidad** establece que, dentro de un relvar, no pueden existir dos tuplas que coincidan en todos los valores de una clave candidata. Esta propiedad se implementa en SQL mediante la restricción `UNIQUE`, que garantiza que los valores de una columna o conjunto de columnas sean distintos en todas las tuplas del relvar. En Pagila, por ejemplo, la tabla `actor` define `actor_id` como clave primaria, lo que implica unicidad automática:

```sql
-- En Pagila: actor_id es único por definición de PRIMARY KEY
\d actor
-- Resultado:
-- Table "public.actor"
-- Column     | Type                  | Modifiers
-- actor_id   | integer               | not null default nextval('actor_actor_id_seq'::regclass)
-- first_name | character varying(45) | not null
-- last_name  | character varying(45) | not null
-- last_update| timestamp without time zone | default now()
-- Indexes:
--     actor_pkey PRIMARY KEY, btree (actor_id)
```

Sin embargo, la unicidad puede aplicarse también a atributos no clave cuando la semántica del dominio lo requiere. Por ejemplo, si se deseara garantizar que no existan dos actores con el mismo nombre completo, se podría añadir una restricción `UNIQUE` sobre la combinación `(first_name, last_name)`:

```sql
ALTER TABLE actor ADD CONSTRAINT unique_actor_name UNIQUE (first_name, last_name);
```

El tratamiento de **valores nulos** constituye otro aspecto crítico del modelo relacional. Date (2019) advierte que "el valor nulo representa la ausencia de información conocida; su presencia introduce lógica trivaluada que complica la evaluación de predicados y la aplicación de operadores algebraicos" (p. 156). En SQL, los valores nulos se representan mediante la palabra clave `NULL` y se manejan con operadores específicos (`IS NULL`, `IS NOT NULL`). La restricción `NOT NULL` prohíbe explícitamente la asignación de valores nulos a un atributo, reforzando la integridad de entidad.

En Pagila, la mayoría de los atributos clave y descriptivos se declaran como `NOT NULL` para garantizar la completitud de la información. Por ejemplo, en la tabla `film`:

```sql
-- Atributos NOT NULL en film
title          character varying(255) NOT NULL,
rental_duration smallint NOT NULL,
rental_rate    numeric(4,2) NOT NULL,
replacement_cost numeric(5,2) NOT NULL,
rating         mpaa_rating DEFAULT 'G',
last_update    timestamp without time zone DEFAULT now() NOT NULL
```

La columna `description`, en cambio, permite valores nulos, reflejando que no todas las películas tienen una sinopsis registrada. Esta flexibilidad semántica debe documentarse explícitamente en el diccionario de datos para evitar interpretaciones erróneas durante el análisis o la migración.

#### 2.3 Verificación de Datos con Restricciones CHECK

Las restricciones `CHECK` permiten definir condiciones lógicas que deben cumplirse para que una operación de inserción o actualización sea aceptada. Date (2019) señala que "las restricciones de dominio, implementadas mediante `CHECK`, son la manifestación operativa de las reglas de negocio en la estructura de datos" (p. 112). En PostgreSQL, estas restricciones se evalúan antes de cada modificación, garantizando que solo los valores válidos persistan en el relvar.

En Pagila, se emplean restricciones `CHECK` para validar rangos y conjuntos de valores permitidos. Por ejemplo, la columna `release_year` en la tabla `film` está restringida a valores entre 1901 y 2155:

```sql
-- Restricción CHECK en film.release_year
CHECK (release_year BETWEEN 1901 AND 2155)
```

Esta restricción previene la inserción de fechas históricamente inválidas o futuristas extremas, preservando la consistencia semántica del catálogo cinematográfico. De manera similar, la clasificación `rating` se limita a un conjunto predefinido mediante el tipo de dominio `mpaa_rating`, que internamente implementa una restricción `CHECK`:

```sql
-- Tipo de dominio mpaa_rating (definido en pagila-schema.sql)
CREATE DOMAIN mpaa_rating AS VARCHAR(10)
CHECK (VALUE IN ('G','PG','PG-13','R','NC-17'));
```

La combinación de `NOT NULL`, `UNIQUE` y `CHECK` constituye el núcleo de la validación de datos en el nivel de esquema, reduciendo la dependencia de lógica de aplicación para mantener la integridad. Sin embargo, es fundamental documentar cada restricción y su justificación semántica, facilitando así la evolución controlada del modelo ante cambios en los requisitos del dominio.

#### 3. Transformación del Diseño Conceptual al Esquema Lógico

El diseño de bases de datos no comienza con la escritura de sentencias SQL, sino con la comprensión del dominio de aplicación y la representación de su estructura semántica mediante un modelo conceptual. El diagrama Entidad-Relación (ER), propuesto por Peter Chen en 1976, constituye la herramienta más extendida para esta fase inicial. Bagui y Earp (2012) enfatizan que el modelo ER no es un lenguaje de implementación, sino un artefacto de comunicación que permite a analistas, diseñadores y usuarios establecer un vocabulario común antes de comprometer recursos computacionales. La transformación de un diagrama ER a un esquema relacional sigue reglas canónicas que garantizan la preservación de la semántica original y la adhesión a los principios de normalización.

La **regla de mapeo para entidades fuertes** establece que cada entidad se convierte en un relvar, cuyos atributos simples se transforman en columnas y cuya clave se convierte en la clave primaria del relvar (Bagui & Earp, 2012). Los **atributos compuestos** se descomponen en sus componentes atómicos, conservando únicamente las hojas de la jerarquía como columnas independientes. Los **atributos multivaluados**, por su parte, generan un relvar nuevo que incluye la clave de la entidad propietaria como clave foránea y el atributo multivaluado como columna; la clave primaria de este nuevo relvar es la concatenación de ambos, garantizando la unicidad de cada combinación (Bagui & Earp, 2012). Esta regla es fundamental para mantener la 1NF, ya que evita el almacenamiento de conjuntos o listas en una sola celda.

Las **relaciones binarias** se mapean según su cardinalidad estructural. En una relación **uno a muchos (1:N)**, la clave del lado "uno" se incorpora como clave foránea en el relvar del lado "muchos". Si la participación del lado "muchos" es total, la clave foránea se declara como `NOT NULL`, reflejando la obligatoriedad semántica de la asociación (Bagui & Earp, 2012). En una relación **muchos a muchos (M:N)**, se crea un relvar de asociación que incluye las claves de ambas entidades como claves foráneas, junto con cualquier atributo propio de la relación. La clave primaria de este relvar es la concatenación de las dos claves foráneas, lo que garantiza que cada combinación de ocurrencias sea única y evita la redundancia (Bagui & Earp, 2012). Las **entidades débiles** requieren un tratamiento especial: se transforman en un relvar que incluye sus atributos, la clave de la entidad identificadora como clave foránea y su clave parcial; la clave primaria del relvar débil es la concatenación de la clave foránea y la clave parcial, asegurando que la existencia de la entidad débil esté supeditada a la entidad fuerte (Bagui & Earp, 2012).

Mata-Toledo y Cushman (2000) complementan este marco metodológico al destacar que el mapeo automático no sustituye el juicio del diseñador. Cada regla debe evaluarse en contexto: volumen de datos, patrones de consulta, requisitos de rendimiento y evolución esperada del sistema. Por ejemplo, una relación 1:N podría resolverse mediante una tabla intermedia si se anticipa que la cardinalidad evolucionará hacia M:N, o si se requiere almacenar metadatos de la asociación que no pertenecen naturalmente a ninguno de los extremos. Asimismo, la selección de claves primarias sustitutas (surrogate keys) versus claves naturales debe ponderarse considerando la estabilidad de los identificadores, el ancho de los índices y la legibilidad para los usuarios finales.

La validación cruzada entre el diagrama ER original y el esquema relacional resultante es un ejercicio fundamental de auditoría estructural. Cada entidad debe corresponder a un relvar, cada relación debe materializarse mediante claves foráneas o tablas de asociación, y cada restricción de participación debe reflejarse en cláusulas `NOT NULL` o restricciones de integridad referencial. La omisión de este paso puede generar esquemas que, aunque funcionales, violen principios de normalización o introduzcan anomalías operativas que comprometan la consistencia de los datos a largo plazo.

#### 4. Arquitectura Modular de SQL: Sublenguajes y Correspondencia Teórica

SQL no es un lenguaje monolítico; su especificación ISO/IEC 9075 lo estructura en componentes funcionales diferenciados que permiten separar responsabilidades entre administradores de bases de datos, diseñadores de esquemas y desarrolladores de aplicaciones. Esta modularidad refleja la distinción teórica entre la definición de estructuras, la manipulación de valores, el control de acceso y la garantía de consistencia transaccional. Mata-Toledo y Cushman (2000) clasifican los sublenguajes principales de SQL en cuatro categorías: DDL (Data Definition Language), DML (Data Manipulation Language), DCL (Data Control Language) y TCL (Transaction Control Language). Cada uno cumple una función específica y se corresponde con operadores o principios del modelo relacional.

El **DDL** permite crear, modificar y eliminar objetos estructurales como bases de datos, esquemas, tablas, índices y restricciones. Date (2019) señala que el DDL materializa el diseño lógico en estructuras físicas, y que cada sentencia DDL modifica el catálogo del sistema dentro de una transacción implícita, garantizando que la definición del esquema sea atómica y consistente. En PostgreSQL, la sintaxis `CREATE TABLE` no solo declara columnas y tipos de datos, sino que integra restricciones de dominio (`CHECK`), unicidad (`UNIQUE`), no nulidad (`NOT NULL`) e integridad referencial (`FOREIGN KEY`). Estas restricciones son la implementación concreta de los principios de integridad de entidad y referencial discutidos en el marco teórico.

El **DML** constituye la interfaz principal para interactuar con los datos almacenados. Aunque incluye operaciones de escritura (`INSERT`, `UPDATE`, `DELETE`), en el contexto académico y analítico la sentencia `SELECT` es la piedra angular. Date (2019) explica que el DML implementa los operadores del álgebra relacional: selección (`WHERE`), proyección (`SELECT`), unión (`JOIN`), diferencia (`EXCEPT`) y producto cartesiano filtrado. Cada consulta SQL puede interpretarse como una expresión algebraica que transforma relvars de entrada en una relación de salida. El principio de sustitución, formulado por Date (2019), establece que "si dos expresiones representan el mismo valor relacional, entonces son intercambiables en cualquier contexto sin alterar el significado del resultado" (p. 43). Este principio garantiza la equivalencia semántica entre diferentes formulaciones de una misma consulta.

El **DCL** gestiona el acceso, roles y privilegios. PostgreSQL implementa un modelo basado en roles que permite asignar permisos a nivel de base de datos, esquema, tabla, columna o incluso fila. Mata-Toledo y Cushman (2000) destacan que el DCL permite asignar y revocar privilegios sobre relvars a roles de usuario, implementando el principio de mínimo privilegio (p. 103). En entornos académicos, la configuración de roles diferenciados (`lectura`, `escritura`, `administrador`) no solo es una práctica de seguridad, sino una manifestación de la separación de responsabilidades en la arquitectura de sistemas de información.

El **TCL** garantiza atomicidad y consistencia mediante las sentencias `BEGIN`, `COMMIT`, `ROLLBACK` y `SAVEPOINT`. Mata-Toledo y Cushman (2000) explican que el TCL permite delimitar transacciones, asegurando las propiedades ACID (Atomicidad, Consistencia, Aislamiento y Durabilidad) (p. 105). En el contexto de bases de datos relacionales, la atomicidad es esencial para operaciones que involucran múltiples relvars, como la inserción de una renta y la actualización del inventario disponible. La omisión de control transaccional explícito puede generar estados inconsistentes que violen la integridad referencial o dejen datos huérfanos.

#### 5. Programación en la Base de Datos: Funciones, Triggers y Stored Procedures

Más allá de la definición estructural y la manipulación básica de datos, los SGBD modernos como PostgreSQL permiten extender su funcionalidad mediante objetos programables: **funciones**, **triggers** y **stored procedures**. Estos constructores permiten encapsular lógica de negocio en el servidor, reduciendo la latencia de red, centralizando la validación y facilitando el mantenimiento. Date (2019) advierte, sin embargo, que "la programación en el servidor debe subordinarse a la preservación de la semántica relacional; la complejidad procedural no debe oscurecer la claridad declarativa del modelo" (p. 389).

#### 5.1 Funciones (Functions)

Una **función** en PostgreSQL es un bloque de código que acepta parámetros de entrada, ejecuta operaciones y retorna un valor o conjunto de valores. Las funciones pueden escribirse en SQL, PL/pgSQL, Python, Perl u otros lenguajes soportados, y se invocan dentro de consultas como expresiones. Su utilidad principal radica en la reutilización de lógica de cálculo, transformación o validación.

**Ejemplo en Pagila:** Supongamos que deseamos calcular la duración estimada de renta en días, considerando que cada película tiene un `rental_duration` en días pero los clientes pueden devolverla antes. Podemos crear una función que aplique una regla de negocio:

```sql
CREATE OR REPLACE FUNCTION calculate_rental_days(
    p_rental_duration SMALLINT,
    p_return_date TIMESTAMP,
    p_rental_date TIMESTAMP
) RETURNS INTEGER AS $$
BEGIN
    -- Si hay fecha de devolución, calcular días reales; si no, usar duración esperada
    IF p_return_date IS NOT NULL THEN
        RETURN EXTRACT(DAY FROM (p_return_date - p_rental_date));
    ELSE
        RETURN p_rental_duration;
    END IF;
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

Esta función puede invocarse en una consulta para enriquecer el análisis de rentas:

```sql
SELECT 
    r.rental_id,
    f.title,
    calculate_rental_days(f.rental_duration, r.return_date, r.rental_date) AS days_used
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
LIMIT 10;
```

#### 5.2 Triggers

Un **trigger** es un mecanismo que ejecuta automáticamente una función en respuesta a eventos específicos (`INSERT`, `UPDATE`, `DELETE`) sobre una tabla. Los triggers son ideales para mantener consistencia derivada, auditar cambios o aplicar reglas de negocio complejas que no pueden expresarse mediante restricciones declarativas.

**Ejemplo en Pagila:** Para mantener un registro histórico de cambios en la tabla `film`, podemos crear un trigger que inserte una copia del registro antiguo en una tabla de auditoría antes de cada actualización:

```sql
-- Tabla de auditoría
CREATE TABLE film_audit (
    audit_id SERIAL PRIMARY KEY,
    film_id INTEGER NOT NULL,
    old_title VARCHAR(255),
    new_title VARCHAR(255),
    changed_by NAME DEFAULT CURRENT_USER,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Función trigger
CREATE OR REPLACE FUNCTION audit_film_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' THEN
        INSERT INTO film_audit (film_id, old_title, new_title)
        VALUES (OLD.film_id, OLD.title, NEW.title);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger asociado
CREATE TRIGGER trg_film_audit
AFTER UPDATE ON film
FOR EACH ROW EXECUTE FUNCTION audit_film_changes();
```

Este trigger garantiza que cada modificación en `film.title` quede registrada con trazabilidad completa, cumpliendo con requisitos de auditoría sin modificar la lógica de aplicación.

#### 5.3 Stored Procedures

Un **stored procedure** es similar a una función, pero está diseñado para ejecutar acciones con efectos secundarios (modificaciones de datos, llamadas a otros procedimientos) y no necesariamente retorna un valor. En PostgreSQL, los procedimientos se declaran con `CREATE PROCEDURE` y se invocan mediante `CALL`.

**Ejemplo en Pagila:** Supongamos que deseamos implementar un procedimiento para procesar devoluciones tardías, aplicando cargos adicionales y actualizando el estado del cliente:

```sql
CREATE OR REPLACE PROCEDURE process_late_return(
    p_rental_id INTEGER,
    p_late_fee NUMERIC(5,2)
) AS $$
DECLARE
    v_customer_id INTEGER;
BEGIN
    -- Obtener el cliente asociado a la renta
    SELECT customer_id INTO v_customer_id FROM rental WHERE rental_id = p_rental_id;
    
    -- Aplicar cargo por retraso
    INSERT INTO payment (customer_id, amount, payment_date)
    VALUES (v_customer_id, p_late_fee, NOW());
    
    -- Actualizar estado de la renta (si existe columna status)
    -- UPDATE rental SET status = 'returned_late' WHERE rental_id = p_rental_id;
    
    COMMIT;
END;
$$ LANGUAGE plpgsql;

-- Invocación
CALL process_late_return(12345, 2.50);
```

Este procedimiento encapsula una regla de negocio compleja (cálculo de cargos, registro de pago, actualización de estado) en una unidad atómica y reutilizable, facilitando la consistencia y el mantenimiento.

#### 6. Estudio de Caso: Análisis Estructural y Operativo de Pagila

La base de datos Pagila constituye un esquema de ejemplo clásico para PostgreSQL, derivado del modelo "DVD Rental" y ampliamente utilizado en entornos académicos para enseñanza, pruebas de rendimiento y desarrollo de consultas complejas. Su diseño refleja principios de normalización avanzados, integridad referencial estricta y una estructura semántica rica que permite ilustrar patrones relacionales fundamentales.

#### 6.1 Estructura del Esquema y Principios de Normalización

Pagila está organizada en torno a entidades como `film`, `actor`, `customer`, `rental`, `inventory`, `category`, `language`, `address`, `city`, `country`, `staff`, `store` y `payment`. La tabla `film` almacena metadatos cinematográficos y está definida con una clave primaria generada automáticamente (`film_id`), restricciones de dominio (`CHECK (release_year BETWEEN 1901 AND 2155)`), valores por defecto (`DEFAULT 3` para `rental_duration`) y tipos especializados (`mpaa_rating` para clasificación, `text[]` para características especiales). La adherencia a la 1NF se garantiza mediante la atomicidad de cada columna; la 2NF y 3NF se preservan al descomponer atributos compuestos y eliminar dependencias transitivas, como la separación de `address` en un relvar independiente vinculado mediante `address_id`.

La relación muchos a muchos entre `film` y `actor` se materializa mediante la tabla intermedia `film_actor`, que contiene únicamente `actor_id` y `film_id` como claves foráneas, junto con una clave primaria compuesta `(actor_id, film_id)`. Este diseño cumple BCNF, ya que la única dependencia funcional no trivial es `{actor_id, film_id} → {last_update}`, y el determinante es la clave primaria. La tabla `film_category` sigue el mismo patrón para la asociación entre películas y categorías, demostrando cómo las relaciones M:N se resuelven sin redundancia y con integridad referencial estricta.

#### 6.2 Ejemplos Ilustrativos por Sublenguaje

**DDL: Definición de Relvars y Restricciones**
```sql
CREATE TABLE film_actor (
    actor_id  smallint NOT NULL REFERENCES actor(actor_id) ON UPDATE CASCADE ON DELETE RESTRICT,
    film_id   smallint NOT NULL REFERENCES film(film_id) ON UPDATE CASCADE ON DELETE RESTRICT,
    last_update timestamp without time zone DEFAULT now() NOT NULL,
    PRIMARY KEY (actor_id, film_id)
);
```
Estas sentencias materializan las reglas de mapeo conceptual: entidades fuertes como relvars, claves primarias automáticas, restricciones de dominio explícitas y claves foráneas con comportamiento propagativo (`ON UPDATE CASCADE`, `ON DELETE RESTRICT`). La cláusula `RESTRICT` previene la eliminación de películas o actores que tengan rentas asociadas, garantizando la integridad referencial.

**DML: Consultas y Álgebra Relacional**
```sql
-- Unión relacional (INNER JOIN) para resolver relación M:N
SELECT a.first_name, a.last_name, f.title
FROM actor a
INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
INNER JOIN film f ON fa.film_id = f.film_id
WHERE a.last_name = 'GUINESS'
LIMIT 10;
```
Esta consulta ilustra la correspondencia entre SQL y álgebra relacional: materializa un producto cartesiano filtrado por condiciones de igualdad (`JOIN ... ON`), aplicando selección (`WHERE`) y proyección (`SELECT`).

**Validación de Integridad**
```sql
-- Verificar que no existan rentas huérfanas (integridad referencial)
SELECT r.rental_id, r.customer_id
FROM rental r
LEFT JOIN customer c ON r.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
-- Resultado esperado: 0 filas
```
Esta consulta sirve como mecanismo de auditoría estructural, validando que la clave foránea `customer_id` en `rental` siempre referencie un cliente existente.

#### 7. Gestión de Respaldo: Mejores Prácticas para Entornos de Producción

La creación y poblamiento de una base de datos no constituye un hito final, sino el inicio de un ciclo de vida operativo donde la preservación de la información se convierte en prioridad estratégica. En entornos de producción, la gestión de respaldos trasciende la mera ejecución de comandos `pg_dump`; se erige como una política de seguridad institucional que exige rigor metodológico, automatización verificable y alineación con marcos normativos. Date (2019) insiste en que "un sistema de base de datos que no implementa mecanismos verificables de recuperación no merece el calificativo de 'confiable'; la integridad lógica debe extenderse a la integridad operacional" (p. 415).

#### 7.1 Tipología de Respaldo en PostgreSQL para Producción

PostgreSQL ofrece utilidades nativas que permiten adaptar la estrategia de respaldo al contexto productivo, considerando objetivos de punto de recuperación (RPO) y tiempo de recuperación (RTO):

| Tipo | Comando | Uso recomendado en producción |
|------|---------|-------------------------------|
| **Respaldo lógico completo** | `pg_dump -Fc -n pagila -f pagila_full.dump` | Migraciones controladas, entornos de staging, recuperación granular de objetos. |
| **Respaldo lógico de esquema** | `pg_dump -s -n pagila -f schema_only.sql` | Control de versiones de estructura, despliegues automatizados, auditoría de cambios. |
| **Respaldo lógico de datos** | `pg_dump -a -n pagila -f data_only.sql` | Repoblamiento de entornos de prueba, anonimización para desarrollo, análisis forense. |
| **Respaldo físico base** | `pg_basebackup -D /backup/base -Fp -Xs -P` | Recuperación ante desastres, clonación de instancias, alta disponibilidad. |
| **Archiving de WAL** | `archive_mode = on` + `archive_command` | Recuperación a punto en tiempo (PITR), replicación asíncrona, auditoría transaccional. |

La selección de la estrategia debe basarse en un análisis de riesgo que considere: criticidad de los datos, frecuencia de cambios, ventanas de mantenimiento aceptables y requisitos regulatorios de retención.

#### 7.2 Componentes de una Política de Respaldo Profesional

Una política de seguridad efectiva para entornos de producción debe integrar los siguientes elementos, alineados con estándares como ISO 27001, NIST SP 800-34 y la Ley Federal de Protección de Datos Personales en Posesión de Particulares (México):

1. **Definición de RPO y RTO**: Establecer objetivos cuantificables de pérdida máxima de datos aceptable (RPO) y tiempo máximo de interrupción tolerable (RTO). Por ejemplo: RPO = 15 minutos (mediante WAL archiving), RTO = 2 horas (mediante restauración automatizada).

2. **Automatización con monitoreo**: Implementar orquestadores como `pgBackRest`, `Barman` o `WAL-G` que gestionen ciclos completos de respaldo, verifiquen integridad mediante checksums SHA-256 y generen alertas ante fallos. Evitar dependencia de scripts manuales o intervención humana.

3. **Retención y rotación escalonada**: Conservar respaldos diarios por 7 días, semanales por 4 semanas, mensuales por 12 meses y anuales por 5 años, eliminando automáticamente versiones obsoletas para cumplir con normativas de protección de datos y optimizar costos de almacenamiento.

4. **Pruebas periódicas de restauración**: Ejecutar simulacros trimestrales de recuperación en entornos aislados, validando no solo la existencia de los archivos de respaldo, sino su funcionalidad operativa. Un respaldo no probado es una ilusión de seguridad.

5. **Almacenamiento geográficamente distribuido**: Mantener copias en ubicaciones físicas o cloud separadas (región distinta), cifradas en tránsito (TLS 1.3) y en reposo (AES-256), accesibles solo mediante autenticación multifactor y roles con privilegios mínimos.

6. **Documentación y auditoría continua**: Registrar metadatos de cada respaldo (fecha, tamaño, hash, responsable, RPO/RTO objetivo) en un sistema de gestión de configuración, alineado con marcos de gobernanza como COBIT o ITIL.

7. **Plan de respuesta ante incidentes**: Integrar los procedimientos de restauración en un plan de continuidad de negocio documentado, con roles definidos, contactos de emergencia y escalación clara ante fallos catastróficos.

#### 7.3 Implementación Práctica en PostgreSQL

A continuación, se presenta un ejemplo de configuración mínima para un entorno de producción con Pagila:

```bash
### 1. Habilitar archiving de WAL en postgresql.conf
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'
wal_level = replica
max_wal_senders = 3

### 2. Configurar pgBackRest para respaldos incrementales
[global]
repo1-path=/backup/pgbackrest
repo1-retention-full=2
repo1-retention-diff=7

[pagila]
pg1-path=/var/lib/postgresql/16/main
pg1-port=5432

### 3. Programar respaldo completo semanal con cron
0 2 * * 0 /usr/bin/pgbackrest --stanza=pagila backup --type=full

### 4. Verificar integridad del respaldo más reciente
pgbackrest --stanza=pagila verify

### 5. Simular restauración en entorno de pruebas (trimestral)
pgbackrest --stanza=pagila --delta --pg1-path=/restore/pagila restore
```

Esta configuración garantiza: (a) recuperación a punto en tiempo mediante WAL, (b) retención escalonada con eliminación automática, (c) verificación de integridad post-respaldo, y (d) capacidad de restauración probada periódicamente.

#### 7.4 Consideraciones Específicas para la Separación Esquema-Datos

La práctica de separar respaldos de esquema y datos, ejemplificada en la distribución de Pagila, ofrece ventajas operativas significativas en producción, pero exige disciplina de gobernanza:

| Ventaja | Requisito de implementación |
|---------|-----------------------------|
| Control de versiones independiente | Integración con Git/GitLab, etiquetado semántico de releases de esquema |
| Despliegue ágil de cambios estructurales | Pipeline CI/CD que valide sintaxis DDL antes de aplicar en producción |
| Anonimización selectiva de datos | Scripts de enmascaramiento documentados y versionados, ejecutados post-restauración |
| Auditoría de cambios en estructura | Logging de DDL mediante `pg_event_trigger` o extensiones como `pgaudit` |

Sin estos controles, la separación física puede generar desincronización entre esquema y datos, violaciones de integridad referencial post-restauración o brechas de seguridad por exposición accidental de información sensible.

#### 8. Conclusiones

La implementación física de un modelo relacional constituye una fase crítica que trasciende la escritura de sentencias SQL; es un ejercicio de disciplina estructural, validación continua y gobernanza operacional. Este documento ha establecido una secuencia rigurosa que parte de la creación del esquema de base de datos como contenedor lógico indispensable, continúa con la definición ordenada de tablas para preservar inquebrantablemente la integridad referencial, y culmina con el poblamiento de datos respetando la cadena de dependencias impuesta por las claves foráneas. La separación entre respaldo de esquema y respaldo de datos, ejemplificada en la distribución de Pagila, se ha analizado como una práctica profesional con ventajas operativas significativas, pero que exige disciplina de sincronización, documentación y verificación.

Se ha enfatizado que la gestión de respaldos en entornos de producción no es una tarea técnica aislada, sino una política de seguridad institucional que requiere definición de RPO/RTO, automatización con monitoreo, retención escalonada, pruebas periódicas de restauración, almacenamiento geográficamente distribuido y documentación auditada. La omisión de estos principios convierte a la base de datos en un activo vulnerable, expuesto a pérdida, corrupción o inconsistencia. Por el contrario, su adopción sistemática garantiza la continuidad operativa, la trazabilidad académica y la formación de profesionales capaces de diseñar, implementar y administrar sistemas relacionales confiables, seguros y evolutivos.

Se recomienda a los estudiantes integrar esta secuencia como estándar operativo en todas las prácticas de la UEA, documentar cada decisión de diseño, validar la integridad referencial mediante consultas de auditoría, y proyectar su aprendizaje hacia técnicas de optimización, particionamiento y gobernanza de datos. La formación en bases de datos relacionales no termina con la ejecución de un script; comienza con la comprensión de por qué cada objeto existe, cómo se relaciona con el dominio y qué implicaciones tiene para la consistencia, seguridad y evolución del sistema. Solo mediante esta integración de teoría, práctica y crítica es posible formar ingenieros de datos preparados para los desafíos tecnológicos del siglo XXI.

#### Referencias

Bagui, S., & Earp, R. (2012). *Database design using entity-relationship diagrams* (2ª ed.). CRC Press.

Date, C. J. (2019). *Database design and relational theory: Normal forms and all that jazz* (2ª ed.). Apress. https://doi.org/10.1007/978-1-4842-5540-7

devrimgunduz. (2026). *Pagila - Sample database for PostgreSQL*. GitHub. https://github.com/devrimgunduz/pagila

Mata-Toledo, R. A., & Cushman, P. K. (2000). *Schaum's outline of fundamentals of relational databases*. McGraw-Hill.

PostgreSQL Global Development Group. (2026). *PostgreSQL 18 documentation*. https://www.postgresql.org/docs/18/

UAM-Iztapalapa. (2026). *Programa analítico de la UEA Bases de Datos (2151106)*. División de Ciencias Básicas e Ingeniería.


