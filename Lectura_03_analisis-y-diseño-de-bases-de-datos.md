---
output:
  word_document: default
  html_document: default
---
# Lectura 03. Diseño Conceptual y Procedimiento de Diseño de Bases de Datos

**Unidad de Enseñanza-Aprendizaje:** Bases de Datos (2151106)  
**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Docente:** Dr. Jesús Zavala Ruiz  
**Modalidad:** Lectura estructurada con exposición guiada  
**Caso de estudio transversal:** Base de datos `pagila` (Esquema *DVD Rental*)  
**Tiempo estimado de dedicación:** 90 minutos (sesión síncrona) + 120 minutos (trabajo autónomo)  
**Última revisión:** 20 de mayo de 2026

---

## Objetivos de Aprendizaje

Al concluir el estudio de esta lectura, el estudiante estará en condiciones de:

1.  Explicar el propósito epistemológico y los límites operativos del modelo Entidad-Relación como representación conceptual de dominios de información.
2.  Identificar y modelar con precisión entidades, atributos, relaciones, cardinalidades y restricciones de participación, empleando el esquema `pagila` como referente estructural validado.
3.  Aplicar un procedimiento sistemático de diseño que articule el levantamiento de requisitos, la construcción del diagrama conceptual, el mapeo al modelo relacional y la validación estructural mediante el entorno `psql`.
4.  Producir un esquema conceptual documentado y trazable, verificando su correspondencia formal con el modelo lógico implementado en PostgreSQL.

## 0. Introducción: La Abstracción como Fundamento del Diseño

El diseño de bases de datos constituye una fase crítica de la ingeniería de software en la cual la complejidad inherente al mundo real se traduce en estructuras de datos formalmente verificables. En este proceso, el modelo Entidad-Relación (ER), propuesto por Peter Chen en 1976, desempeña un papel fundamental no como lenguaje de implementación, sino como **artefacto de comunicación y abstracción semántica** (Chen, 1976). Su valor reside en permitir que analistas, administradores de datos y desarrolladores establezcan un vocabulario compartido antes de comprometer recursos computacionales en la materialización física del esquema.

La presente sesión tiene como propósito examinar los fundamentos teóricos del modelo conceptual y presentar un procedimiento de diseño iterativo, riguroso y trazable. Como hilo conductor pedagógico, se empleará la base de datos `pagila`, disponible en el entorno de laboratorio desarrollado en la Fase 2 del curso. Este esquema, ampliamente utilizado en contextos académicos e industriales, ejemplifica de manera concreta cómo un dominio de negocio —la gestión de renta de contenidos audiovisuales— se descompone en elementos estructurales antes de ser materializado en tablas PostgreSQL. A través de este caso de estudio, se contrastará la teoría del modelado con una implementación relacional operativa, facilitando así la comprensión de los principios de diseño que subyacen a sistemas de información reales.

## 1. Tema 3.1: El Modelo Entidad-Relación como Herramienta de Abstracción Conceptual

### 1.1. Definición y Propósito Epistemológico

El modelo Entidad-Relación es una herramienta de **modelado conceptual** cuya finalidad es representar la estructura semántica de un dominio de discurso mediante constructores gráficos formalmente definidos. Su objetivo primordial no es describir cómo se almacenarán los datos, sino capturar **qué entidades existen** en el dominio y **de qué manera se relacionan** entre sí, con independencia de la tecnología de almacenamiento subyacente (Elmasri & Navathe, 2016).

En el contexto del laboratorio, el esquema `pagila` ilustra de manera paradigmática este principio: el dominio de negocio de una empresa de renta de películas se descompone en entidades como `film`, `actor`, `customer` y `rental`, cada una con sus atributos descriptivos y sus relaciones semánticas, antes de ser traducido a sentencias `CREATE TABLE` en PostgreSQL. Esta separación entre el nivel conceptual y el nivel lógico es esencial para garantizar que el diseño refleje fielmente los requisitos del negocio, sin verse distorsionado por consideraciones prematuras de implementación.

### 1.2. Constructores Fundamentales y su Manifestación en `pagila`

El modelo ER se articula en torno a tres constructores primarios, los cuales encuentran correspondencia directa en el esquema relacional de `pagila`:

| Constructor | Definición formal | Ejemplo en `pagila` | Materialización en PostgreSQL |
|-------------|-------------------|---------------------|-------------------------------|
| **Entidad** | Objeto distinguible del dominio con existencia propia e identidad única | `film`, `actor`, `customer`, `rental` | `CREATE TABLE film (...)` con clave primaria |
| **Atributo** | Propiedad descriptiva que caracteriza a una entidad o relación | `title`, `release_year`, `rental_rate`, `first_name` | Columnas tipadas (`VARCHAR`, `INTEGER`, `NUMERIC`, etc.) |
| **Relación** | Asociación semántica que vincula dos o más entidades | `actúa_en`, `alquila`, `posee_inventario` | Claves foráneas (`FOREIGN KEY`) o tablas intermedias asociativas |

### 1.3. Clasificación de Atributos en el Contexto de `pagila`

La correcta identificación del tipo de atributo es esencial para garantizar la fidelidad del modelo conceptual. En `pagila`, observamos los siguientes casos:

- **Atributos simples:** Aquellos que no admiten descomposición significativa. Ejemplos: `film.rating`, `actor.last_name`.
- **Atributos compuestos:** Aquellos que pueden descomponerse en subcomponentes con significado propio. En `pagila`, la entidad `address` se modela mediante múltiples atributos simples (`address`, `address2`, `district`, `city_id`, `postal_code`, `phone`), reflejando una descomposición explícita en el nivel lógico.
- **Atributos derivados:** Aquellos cuyo valor puede calcularse a partir de otros atributos. Por ejemplo, `rental.duration_days` podría derivarse de la expresión `return_date - rental_date`. En el diseño conceptual, estos atributos suelen omitirse o documentarse como notas, dado que no requieren almacenamiento físico.
- **Atributos multivaluados:** Aquellos que pueden asumir múltiples valores para una misma ocurrencia de entidad. En `pagila`, `film.special_features` se almacena como un arreglo `text[]` en PostgreSQL; sin embargo, en el nivel conceptual ER, se modela como atributo multivaluado, lo cual implica, en el mapeo relacional, la creación de una tabla independiente si se desea preservar la normalización.
- **Atributos clave (identificadores):** Aquellos que garantizan la unicidad de cada ocurrencia de entidad. Ejemplos: `film.film_id`, `actor.actor_id`. En el modelo relacional, se materializan como `PRIMARY KEY`.

### 1.4. Cardinalidad y Participación: Restricciones Estructurales del Dominio

La cardinalidad y la participación son restricciones que expresan reglas de negocio en el nivel conceptual. Su correcta especificación es determinante para el mapeo al modelo relacional.

**Cardinalidad estructural** indica el número máximo de ocurrencias de una entidad que pueden asociarse con una ocurrencia de otra entidad. En `pagila`, identificamos los siguientes patrones:

- `actor` (M) `──` `actúa_en` `──` (N) `film`: Relación muchos a muchos. Un actor puede participar en múltiples películas; una película puede contar con múltiples actores.
- `customer` (1) `──` `realiza` `──` (N) `rental`: Relación uno a muchos. Un cliente puede generar múltiples rentas; cada renta pertenece a un único cliente.
- `film` (1) `──` `clasifica_en` `──` (N) `category`: Relación muchos a muchos resuelta mediante la tabla intermedia `film_category`.

**Participación** indica si la asociación entre entidades es obligatoria (`total`) u opcional (`parcial`). En `pagila`, la relación entre `rental` e `inventory` es total para `rental` (no puede existir un registro de renta sin referenciar un inventario válido), pero parcial para `inventory` (un artículo de inventario puede existir sin haber sido rentado aún).

> **Nota:** La cardinalidad no es una propiedad técnica del sistema gestor de bases de datos, sino una restricción semántica del dominio. Su correcta especificación en el nivel conceptual determina la ubicación de las claves foráneas y la necesidad de tablas asociativas en el nivel lógico.

### 1.5. Limitaciones Inherentes al Modelo ER

A pesar de su utilidad, el modelo ER presenta limitaciones que el diseñador debe reconocer:

1.  No permite expresar restricciones de dominio complejas, como verificaciones de rango o enumeraciones (ej. `CHECK (rating IN ('G','PG','PG-13','R','NC-17'))`).
2.  No representa comportamiento, transacciones ni reglas de negocio procedimentales; su ámbito es estrictamente estructural.
3.  Requiere una transformación formal rigurosa para garantizar la normalización relacional y la integridad referencial explícita en el nivel lógico.

Estas limitaciones subrayan la necesidad de complementar el modelo conceptual con documentación adicional y con una fase de validación en el entorno de implementación.

## 2. Tema 3.2: Procedimiento Sistemático de Diseño de Bases de Datos

### 2.1. Marco Metodológico Iterativo

El diseño de bases de datos se estructura en fases secuenciales pero inherentemente iterativas, conforme a los principios de la ingeniería de software (Elmasri & Navathe, 2016):

1.  **Recolección y análisis de requisitos:** Identificación de actores, procesos y artefactos de información.
2.  **Diseño conceptual:** Construcción del modelo ER independiente de tecnología.
3.  **Diseño lógico:** Mapeo del modelo ER al modelo relacional.
4.  **Diseño físico:** Definición de índices, estrategias de particionamiento y parámetros de almacenamiento.
5.  **Validación y refinamiento:** Verificación de integridad, desempeño y adherencia a requisitos.

En esta unidad, se abordarán exhaustivamente las fases 1 a 3, dado que constituyen la base estructural sobre la cual se sustentará la implementación en PostgreSQL. El esquema `pagila` servirá como ejercicio de **ingeniería inversa controlada**, permitiendo al estudiante contrastar sus modelos conceptuales con un esquema relacional ya validado en entornos productivos.

### 2.2. Fase 1: Levantamiento de Requisitos y Diccionario de Datos Preliminar

Esta fase implica la identificación sistemática de:

- **Actores del dominio:** clientes, empleados, administradores.
- **Procesos de negocio:** registro de clientes, gestión de inventario, procesamiento de rentas.
- **Artefactos de información:** películas, actores, categorías, pagos.

Como producto tangible, se elabora un **diccionario de datos preliminar** que catalogue entidades candidatas, sus atributos descriptivos y las reglas de negocio relevantes. Por ejemplo: *"Una película puede tener múltiples actores, y un actor puede participar en múltiples películas"* implica una relación muchos a muchos que deberá resolverse mediante una tabla asociativa en el nivel lógico.

### 2.3. Fase 2: Construcción del Diagrama Conceptual

En esta fase, el diseñador:

1.  Selecciona y depura las entidades, eliminando redundancias semánticas.
2.  Asigna identificadores únicos y valida su unicidad en el dominio.
3.  Define relaciones con cardinalidad y participación explícitas, utilizando notación estándar (Chen o Crow's Foot).
4.  Documenta supuestos, excepciones y restricciones no representables gráficamente.

> **Regla de oro del modelado conceptual:** Un diagrama ER debe ser comprensible para un experto del dominio sin conocimiento técnico en bases de datos. Si requiere explicación extensa, probablemente está sobre-modelado o sub-especificado.

### 2.4. Fase 3: Mapeo al Modelo Relacional

La transformación del modelo ER al modelo relacional sigue patrones formales que garantizan la preservación de la semántica y la integridad estructural:

| Patrón ER | Transformación lógica | Aplicación en `pagila` |
|-----------|----------------------|------------------------|
| Entidad fuerte | → Tabla con clave primaria | `film(film_id PK, title, description, release_year, ...)` |
| Entidad débil | → Tabla con PK compuesta (clave parcial + FK a entidad fuerte) | No aplica directamente en `pagila` |
| Relación 1:1 | → FK en cualquiera de las tablas (preferentemente la de participación total) | `staff` ↔ `store` (simplificado en el esquema) |
| Relación 1:N | → FK en la tabla del lado `N` | `rental.customer_id` FK → `customer.customer_id` |
| Relación M:N | → Nueva tabla asociativa con FKs a ambas entidades + atributos propios | `film_actor(film_id FK, actor_id FK, PRIMARY KEY(film_id, actor_id))` |
| Atributos multivaluados | → Tabla independiente con FK a la entidad original | `film_category` o manejo como arreglo `text[]` en PostgreSQL |

### 2.5. Fase 4: Normalización y Validación Estructural

Una vez obtenido el esquema relacional, se aplican progresivamente las formas normales (1FN, 2FN, 3FN) para eliminar anomalías de inserción, actualización y borrado (Date, 2003). Este proceso implica:

- Verificación de dependencias funcionales y preservación de la semántica original.
- Revisión de consistencia cruzada entre el diagrama ER y el esquema relacional resultante.
- Validación empírica en el entorno de laboratorio: ejecutar comandos como `\d film`, `\d film_actor` y `\d customer` en `psql` para verificar que las claves primarias y foráneas coinciden con el mapeo teórico.

### 2.6. Herramientas y Buenas Prácticas Académicas

Para apoyar el proceso de diseño, se recomiendan las siguientes herramientas:

- **Modelado visual:** ERDPlus, dbdiagram.io, pgModeler, draw.io (con plantillas ER).
- **Convenciones de nomenclatura:** Sustantivos singulares para entidades, verbos para relaciones, `snake_case` para atributos, prefijo `fk_` para claves foráneas.
- **Documentación obligatoria:** Supuestos de diseño, justificación de cardinalidades, lista de restricciones no representables en ER y versión del diagrama.

> **Principio de trazabilidad:** Cada decisión de diseño debe quedar documentada y ser rastreable desde los requisitos hasta la implementación. Esto facilita la auditoría, el mantenimiento y la evolución del esquema.

## 3. Análisis Estructural del Caso `pagila`

El esquema `pagila` constituye un dominio rico en patrones relacionales y resulta ideal para ejercitar la transición del modelo conceptual al lógico. A continuación, se presenta la estructura de análisis que el estudiante deberá replicar en sus ejercicios:

### Entidades Nucleares
`film`, `actor`, `customer`, `rental`, `inventory`, `store`, `staff`, `address`, `city`, `country`, `language`, `category`, `payment`.

### Relaciones Críticas y su Mapeo Relacional

1.  **`actor` ↔ `film` (M:N)**  
    Se resuelve mediante la tabla asociativa `film_actor`, que contiene únicamente las claves foráneas `actor_id` y `film_id`, conformando una clave primaria compuesta. Este patrón garantiza la integridad referencial y evita la redundancia.

2.  **`customer` ↔ `rental` (1:N)**  
    La tabla `rental` almacena `customer_id` como clave foránea, garantizando que cada registro de renta apunte a un cliente válido. La participación total de `rental` hacia `customer` se refleja en la restricción `NOT NULL` de la FK.

3.  **`inventory` ↔ `film` (N:1)**  
    `inventory.film_id` como FK permite rastrear qué copias físicas pertenecen a cada título cinematográfico, facilitando consultas de disponibilidad y gestión de stock.

4.  **`address` ↔ `city` ↔ `country` (Jerarquía N:1:N:1)**  
    Esta normalización geográfica evita redundancia y facilita consultas agregadas por región, país o distrito, ilustrando el principio de diseño "una vez, un solo lugar" (*once and only once*).

### Validación mediante Consultas del Laboratorio

Las consultas ejecutadas en la Fase 2 del laboratorio no constituyen meros ejercicios sintácticos, sino **pruebas de integridad estructural** derivadas del diseño conceptual:

- Un `JOIN` entre `actor`, `film_actor` y `film` valida la correcta resolución de la relación M:N.
- `COUNT(*) FROM film` confirma la cardinalidad mínima esperada del dominio.
- `ILIKE '%dragon%'` demuestra que los atributos textuales están adecuadamente tipados e indexables para búsquedas semánticas.

> **Reflexión crítica:** La capacidad de traducir una restricción conceptual (ej. "un actor participa en muchas películas") en una estructura relacional verificable (`film_actor` con PK compuesta) es una competencia fundamental del diseñador de bases de datos.

## 4. Cierre

### Síntesis Conceptual

El modelo Entidad-Relación es, ante todo, un artefacto de comunicación que permite alinear el entendimiento entre expertos del dominio y especialistas técnicos. El procedimiento de diseño, por su parte, constituye un mecanismo de control de calidad estructural que garantiza la trazabilidad desde los requisitos hasta la implementación. El esquema `pagila` ejemplifica de manera paradigmática cómo un modelo conceptual bien fundamentado se traduce en un esquema relacional normalizado, seguro y eficiente.

### Entregable Esperado

Como producto de esta sesión, el estudiante deberá entregar:

1.  Diagrama ER del esquema `pagila` en notación estándar (Chen o Crow's Foot).
2.  Tabla de mapeo relacional que justifique la ubicación de cada clave foránea y la resolución de relaciones M:N.
3.  Documento de supuestos de diseño (máximo 3 páginas), incluyendo: reglas de negocio no representables en ER, decisiones de cardinalidad y participación, y referencias a consultas de validación en `psql`.

### Rúbrica de Evaluación

| Criterio | Ponderación |
|----------|-------------|
| Coherencia semántica entre dominio y modelo | 30% |
| Precisión en la especificación de cardinalidades y participación | 25% |
| Corrección formal del mapeo al modelo relacional | 25% |
| Calidad de la documentación, nomenclatura y trazabilidad | 20% |

### Lectura Complementaria

- Elmasri, R., & Navathe, S. B. (2016). *Fundamentals of Database Systems* (7.ª ed.). Pearson. Capítulos 7-8.
- Chen, P. P.-S. (1976). The Entity-Relationship Model: Toward a Unified View of Data. *ACM Transactions on Database Systems*, 1(1), 9–36. https://doi.org/10.1145/320434.320440

### Notas para la Práctica Autónoma

1.  **Validación empírica:** Utilice la sesión de laboratorio para ejecutar `\d film`, `\d actor` y `\d film_actor` en `psql`. Observe cómo el modelo lógico materializa las decisiones de diseño conceptual mediante claves primarias, foráneas y restricciones de integridad.

2.  **Énfasis conceptual:** Recuerde que la cardinalidad no es una propiedad técnica del SGBD, sino una restricción semántica del dominio. `pagila` lo ilustra claramente al diferenciar entre entidades de negocio (`film`, `customer`) y entidades de asociación (`film_actor`, `rental`).

3.  **Ejercicio de descomposición:** Si presenta dificultades con relaciones M:N, proponga el ejercicio de descomposición manual sobre `actor` ↔ `film` antes de recurrir a herramientas automáticas. Escriba a mano la tabla `film_actor` con sus columnas y restricciones; luego verifíquela contra el esquema real.

4.  **Responsabilidad del diseñador:** PostgreSQL no interpreta diagramas ER; la transformación lógica es responsabilidad exclusiva del diseñador. La validación cruzada entre el diagrama conceptual y las tablas reales constituye un ejercicio fundamental de auditoría estructural y calidad de diseño.

5.  **Bitácora de decisiones:** Mantenga un registro escrito de cada decisión de modelado. Esta práctica no solo facilita la evaluación, sino que desarrolla la disciplina documental requerida en entornos profesionales de ingeniería de software.

> *"El modelo de datos es un contrato entre el dominio del problema y la solución técnica. Su claridad determina la robustez del sistema."* — Adaptado de Date (2003).


### Referencias

Chen, P. P.-S. (1976). The Entity-Relationship Model: Toward a Unified View of Data. *ACM Transactions on Database Systems*, 1(1), 9–36. https://doi.org/10.1145/320434.320440

Date, C. J. (2003). *An Introduction to Database Systems* (8.ª ed.). Addison-Wesley.

Elmasri, R., & Navathe, S. B. (2016). *Fundamentals of Database Systems* (7.ª ed.). Pearson.