### Laboratorio 09. Ejercicios Prácticos de Consolidación de Diseño de Bases de Datos
### Validación, Portabilidad e Ingeniería Inversa en el Diseño de Bases de Datos

**dr. Jesús Zavala Ruiz**  
**Creación:** Junio de 2026)  

---

#### Introducción

En mi opinión, el diseño de bases de datos no culmina con la obtención del diagrama del modelo relacional o la generación del script SQL. La verdadera prueba de fuego de cualquier modelo conceptual reside en su capacidad para materializarse en un sistema físico robusto, portable y semánticamente fiel a la visión del negocio. Los ejercicios que se presentan a continuación no son meras prácticas técnicas; son ejercicios de **validación epistemológica** que permiten al estudiante confrontar la teoría con la praxis, descubriendo las tensiones entre la estandarización ANSI y las particularidades de cada motor de base de datos, así como las limitaciones y potencialidades de las herramientas de inteligencia artificial en la ingeniería inversa.

Estos ejercicios cierran el ciclo metodológico de los Seis Pasos de Captain, llevando al estudiante desde la implementación física hasta la validación empírica y la transformación entre dialectos, demostrando que el diseño de bases de datos es un proceso iterativo, crítico y reflexivo.

#### Ejercicio 1: Implantación y Validación de la Base de Datos Académica

##### 1.1 Objetivo Pedagógico

Validar la corrección sintáctica, la integridad referencial y la portabilidad de los scripts SQL-92/SQL:2016 y PostgreSQL 16+ desarrollados en el Paso 6 del Estudio de Caso (Zavala Ruiz, 2026). Este ejercicio permite al estudiante confrontar la teoría de la normalización con la realidad de la implementación física, verificando que las restricciones `CHECK`, `UNIQUE`, `FOREIGN KEY` y las claves subrogadas `GENERATED ALWAYS AS IDENTITY` funcionen según lo diseñado.

##### 1.2 Contexto Metodológico

Como señala Captain (2015, p. 200), la implementación física es el momento en que el modelo conceptual se somete a las rigideces del motor de base de datos. La estandarización SQL-92/SQL:2016 promete portabilidad, pero en la práctica, cada RDBMS posee particularidades sintácticas y de comportamiento que deben ser validadas empíricamente. Este ejercicio no es solo "ejecutar un script"; es un ejercicio de **verificación de la integridad de dominio, entidad y referencial** que garantizamos teóricamente en los Pasos 1 a 5.

##### 1.3 Instrucciones Paso a Paso

**Fase 1: Preparación del Entorno**

1. Instalar un Sistema Gestor de Bases de Datos Relacional que cumpla con el estándar SQL-92/SQL:2016 (ej. Oracle Database Express, IBM DB2, SQL Server Developer o PostgreSQL).
2. Instalar PostgreSQL 16+ en un entorno local o en el laboratorio.
3. Crear dos bases de datos vacías: `registro_academico_sql92` y `registro_academico_pg16`.

**Fase 2: Ejecución de los Scripts**

4. Ejecutar el script **SQL-92/SQL:2016** (Sección 6.9) en la base de datos `registro_academico_sql92`. Documentar cualquier error o advertencia.
5. Ejecutar el script **PostgreSQL 16+** (Sección 6.10) en la base de datos `registro_academico_pg16`. Documentar cualquier error o advertencia.

**Fase 3: Validación Estructural**

6. Verificar que las 11 tablas fueron creadas en el orden topológico correcto.
7. Verificar que todas las claves primarias (PK) y claves foráneas (FK) fueron creadas correctamente mediante consultas al catálogo del sistema (ej. `INFORMATION_SCHEMA.TABLE_CONSTRAINTS`).
8. Verificar que los índices explícitos fueron creados en las columnas especificadas.

**Fase 4: Pruebas de Integridad (El "Torture Test")**

9. **Prueba de Integridad de Dominio:** Intentar insertar un registro en `student` con un `gender` que no sea 'MALE' o 'FEMALE' (si se agregó la restricción) o con una fecha de nacimiento futura. Verificar que el RDBMS rechace la operación.
10. **Prueba de Integridad de Entidad:** Intentar insertar dos estudiantes con el mismo `ss_number`. Verificar que la restricción `UNIQUE` actúe.
11. **Prueba de Integridad Referencial:** Intentar insertar una `scheduled_class` con un `semester_id` que no exista en la tabla `semester`. Verificar que la FK rechace la operación.
12. **Prueba de Cascada:** Actualizar el `user_id` de un usuario en la tabla `app_user` y verificar si las FKs con `ON UPDATE CASCADE` actualizan automáticamente las referencias en `student` y `lecturer`.
13. **Prueba de Restricción:** Intentar eliminar un `course` que tenga `scheduled_classes` asociadas. Verificar que `ON DELETE RESTRICT` impida la eliminación.

**Fase 5: Inserción de Datos de Prueba**

14. Insertar datos de prueba coherentes: al menos 3 semestres, 5 cursos, 5 docentes, 10 estudiantes, 10 clases programadas, 20 inscripciones, y registros de actividad.
15. Ejecutar consultas `JOIN` complejas para verificar que las relaciones funcionan correctamente (ej. "Listar el nombre del estudiante, el nombre del curso, el horario y el nombre del docente para todas las inscripciones del Semestre 2026-1").

##### 1.4 Entregables Esperados

- Un informe técnico que documente:  
  - Las diferencias sintácticas o de comportamiento encontradas entre el script SQL-92 y el script PostgreSQL 16+.  
  - Los resultados de las pruebas de integridad (capturas de pantalla de los errores esperados).  
  - Las consultas `JOIN` ejecutadas y sus resultados.  
  - Una reflexión crítica sobre la portabilidad real del estándar SQL-92.  

##### 1.5 Reflexión Final

En mi opinión, este ejercicio demuestra que la estandarización SQL es un ideal normativo, no una realidad absoluta. Las particularidades de cada motor (como el manejo de `IDENTITY`, los tipos de datos `TEXT` vs `CLOB`, o el plegado de mayúsculas/minúsculas en PostgreSQL) obligan al diseñador a conocer profundamente la herramienta que utilizará. La normalización no es solo una regla teórica; es un contrato de integridad que el RDBMS debe hacer cumplir.

#### Ejercicio 2: Ingeniería Inversa Asistida por IA - De Sakila a Pagila

##### 2.1 Objetivo Pedagógico

Utilizar herramientas de Inteligencia Artificial (IA) generativa para realizar un ejercicio de **ingeniería inversa y transformación de dialectos SQL**, partiendo del script de creación de la base de datos Sakila (diseñada para MySQL) y transformándolo en:
1. Un script estandarizado en SQL-92/SQL:2016.  
2. Un script optimizado para MariaDB 10+.  
3. Un script optimizado para PostgreSQL 16+ (conocido como Pagila).  

Este ejercicio permite al estudiante evaluar las capacidades y limitaciones de la IA en la traducción entre dialectos SQL, así como reflexionar sobre la pérdida de semántica en la ingeniería inversa.

##### 2.2 Contexto Metodológico

La **ingeniería inversa** es el proceso de reconstruir un modelo o script a partir de una implementación existente. En el contexto de las bases de datos, esto implica transformar un script de un dialecto SQL a otro o reconstruir el modelo conceptual a partir del modelo físico. La irrupción de la IA generativa ha prometido automatizar este proceso, pero, ¿hasta qué punto la IA comprende la **semántica** de las reglas de negocio o solo traduce la **sintaxis**? Este ejercicio busca responder esa pregunta mediante un caso real: la migración de Sakila (MySQL) a Pagila (PostgreSQL), un caso clásico en la comunidad de bases de datos.

##### 2.3 Instrucciones Paso a Paso

**Fase 1: Obtención del Script Original**

1. Descargar el script oficial de creación de la base de datos Sakila para MySQL desde el repositorio oficial: `https://dev.mysql.com/doc/index-other.html` (archivos `sakila-schema.sql` y `sakila-data.sql`).  
2. Analizar el script `sakila-schema.sql` e identificar:  
   - Las particularidades sintácticas de MySQL (ej. `AUTO_INCREMENT`, `ENUM`, `SET`, `ENGINE=InnoDB`, `unsigned`, `ZEROFILL`).  
   - Las relaciones entre tablas (identificar las FK).  
   - Las restricciones `CHECK` (si las hay) y los disparadores (`TRIGGER`).  

**Fase 2: Transformación a SQL-92/SQL:2016 con IA**

3. Utilizar una herramienta de IA generativa (ej. ChatGPT, Claude, Gemini) con el siguiente *prompt*:
   > "Actúa como un arquitecto de bases de datos experto en estándares ANSI. Toma el siguiente script de creación de tablas para MySQL (Sakila) y transfórmalo en un script estrictamente compatible con el estándar SQL-92/SQL:2016. Elimina todas las extensiones propietarias de MySQL (como `AUTO_INCREMENT`, `ENUM`, `ENGINE`, `unsigned`). Reemplaza `AUTO_INCREMENT` con `GENERATED ALWAYS AS IDENTITY`. Convierte los tipos `ENUM` y `SET` en `VARCHAR` con restricciones `CHECK` si es necesario. Mantén la integridad referencial y la semántica de las reglas de negocio. No incluyas datos, solo la estructura."
   
4. Proporcionar a la IA el contenido de `sakila-schema.sql` y generar el script SQL-92 resultante.
5. **Validación Humana:** Revisar críticamente el script generado por la IA. Identificar errores, omisiones o traducciones semánticamente incorrectas (ej. ¿La IA preservó la restricción de que un `rental_date` debe ser anterior a un `return_date`?).

**Fase 3: Generación de Scripts para MariaDB y PostgreSQL (Pagila)**

6. **Para MariaDB 10+:** Utilizar la IA con el siguiente *prompt*:
   > "Toma el script SQL-92 generado y adáptalo para MariaDB 10+. MariaDB es compatible con MySQL pero tiene extensiones propias. Optimiza el script para MariaDB, manteniendo `AUTO_INCREMENT` (ya que es compatible), pero asegurando que las restricciones `CHECK` sean soportadas (MariaDB 10.2+ las soporta). Añade comentarios explicativos sobre las diferencias con MySQL."

7. **Para PostgreSQL 16+ (Pagila):** Utilizar la IA con el siguiente *prompt*:
   > "Toma el script SQL-92 generado y adáptalo para PostgreSQL 16+. Reemplaza `GENERATED ALWAYS AS IDENTITY` si es necesario (aunque PostgreSQL 10+ lo soporta). Convierte los tipos `ENUM` de MySQL en tipos `ENUM` nativos de PostgreSQL (`CREATE TYPE ... AS ENUM`). Reemplaza `DATETIME` por `TIMESTAMP`. Convierte `TINYINT` en `SMALLINT`. Añade restricciones `CHECK` donde sea pertinente. Optimiza para PostgreSQL, considerando su manejo de `TEXT`, `BOOLEAN`, y secuencias."

8. **Validación Humana:** Revisar críticamente ambos scripts generados. Verificar que las particularidades de cada motor sean correctamente aplicadas.

**Fase 4: Pruebas de Implantación**

9. Instalar MySQL 8+, MariaDB 10+, y PostgreSQL 16+.
10. Ejecutar los tres scripts (MySQL original, MariaDB generado, PostgreSQL generado) y documentar los errores encontrados.
11. Iterar con la IA para corregir los errores, proporcionando los mensajes de error del RDBMS como contexto.

##### 2.4 Entregables Esperados

- Un informe comparativo que incluya:
  - El script SQL-92 generado por la IA (con las correcciones humanas).  
  - El script para MariaDB generado por la IA.  
  - El script para PostgreSQL 16+ (Pagila) generado por la IA.  
  - Una tabla comparativa de las diferencias sintácticas entre los tres dialectos (MySQL, MariaDB, PostgreSQL).  
  - Un registro de los errores encontrados y cómo se resolvieron (iteraciones con la IA).  
  - Una reflexión crítica sobre la capacidad de la IA para comprender la semántica de las reglas de negocio vs. solo traducir sintaxis.  

##### 2.5 Reflexión Final

En mi opinión, este ejercicio revela una verdad incómoda sobre la IA generativa: es excelente para la traducción sintáctica, pero deficiente en la comprensión semántica. La IA puede convertir `AUTO_INCREMENT` en `GENERATED ALWAYS AS IDENTITY`, pero no puede inferir que un `ENUM('G', 'PG', 'PG-13', 'R', 'NC-17')` en la tabla `film` representa una clasificación de contenido que debería ser una tabla separada (`rating`) para cumplir con la Tercera Forma Normal. La IA traduce, pero no normaliza. El diseñador humano debe mantener el control crítico del proceso, utilizando la IA como un asistente, no como un reemplazo.

#### Conclusión

Los dos ejercicios prácticos presentados no son meras aplicaciones técnicas; son ejercicios de **pensamiento crítico** sobre la naturaleza del diseño de bases de datos. El Ejercicio 1 valida la portabilidad y la integridad de la implementación física frente al estándar; el Ejercicio 2 explora las capacidades y limitaciones de la IA en la ingeniería inversa y la transformación entre dialectos.

En mi opinión, estos ejercicios preparan al estudiante para los desafíos reales de la industria, donde la estandarización es un ideal, la IA es una herramienta (no un reemplazo), y la comprensión profunda del modelo conceptual es la única garantía de un diseño robusto y escalable. Como señala Captain (2015, p. 200), "tenga siempre a mano el Manual de Referencia del RDBMS", pero yo agregaría: tenga siempre presente el modelo conceptual, porque sin él, la implementación física es solo un conjunto de tablas sin alma.

#### Referencias

Captain, F. A. (2015). *Six-step relational database design: A step by step approach to relational database design and development*. Fidel Captain.  

Everest, G. C. (1976). Basic data structure models explained with a common example. In *Computing Systems 1976, Proceedings Fifth Texas Conference on Computing Systems* (pp. 39-46). IEEE Computer Society Publications Office.  

MySQL. (2024). *Sakila Sample Database* [Software]. <https://dev.mysql.com/doc/index-other.html>  

PostgreSQL. (2024). *PostgreSQL 16 Documentation*. <https://www.postgresql.org/docs/16/>  
  

**Dr. Jesús  
