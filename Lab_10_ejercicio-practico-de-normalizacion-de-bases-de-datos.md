### Laboratorio_10. Ejercicio Práctico de Normalización de Bases de Datos

**Dr. Jesús Zavala Ruiz**  
**Creación:** Junio de 2026

---

#### 1. Introducción

El diseño eficiente de bases de datos relacionales es fundamental para garantizar la integridad, el rendimiento y la escalabilidad de los sistemas de información. Una estructura mal diseñada puede derivar en anomalías de inserción, actualización y eliminación, así como en una redundancia de datos que compromete la confiabilidad del sistema.

Este documento presenta un ejercicio práctico y progresivo de normalización, basado en la sección 2.4 de Koseoglu (2025). Al contenido original se le han incorporado cuatro secciones adicionales —Diccionario de Datos, Especificaciones SQL, Conclusiones y — para ofrecer una visión completa que abarca desde el análisis conceptual hasta la implementación técnica. Partiendo de una estructura inicial similar a una hoja de cálculo, se guía al lector a través de las formas normales fundamentales (1FN, 2FN y 3FN) y se incluye un análisis complementario de la Forma Normal de Boyce-Codd (BCNF) para abordar dependencias funcionales más complejas.

#### 2. Fundamentos de la Normalización

Aunque la normalización no es un componente intrínseco del modelo relacional teórico, su aplicación práctica es tan prevalente que resulta inseparable del diseño moderno de bases de datos. Constituye un proceso estructurado que aplica mejores prácticas para organizar los datos, permitiendo obtener un esquema o plano (*blueprint*) optimizado.

Mientras que los desarrolladores experimentados pueden intuir diseños normalizados gracias a su experiencia, las reglas formales de normalización son herramientas invaluables para que los desarrolladores novatos validen sus estructuras. Para comprender este proceso, observaremos su aplicación en un caso concreto. La **Tabla 1** muestra nuestro punto de partida: un registro plano que contiene información sobre conductores, sus gerentes supervisores y los vehículos asignados.

**Tabla 1: Estructura Inicial de Datos de Conductores**
| driver_id | driver_name | manager | manager_ext | car_1 | car_2 | car_3 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1022 | Jordan Lee | Alex Mitchell | 345 | XJ4P2T9 | B9R3L7Q | M6V1H8Z |
| 1023 | Casey Morgan | Jamie Carter | 123 | XJ4P2T9 | K2S8N4R | L1K4P8S |

Esta representación refleja típicamente una hoja de cálculo básica. ¿Puede identificar las deficiencias estructurales? A continuación, las abordaremos paso a paso.

##### 3.1.1 Aplicación de la Primera Forma Normal (1FN)

La **Primera Forma Normal (1FN)** establece que todos los atributos deben contener valores atómicos (indivisibles) y que no debe existir repetición de grupos de columnas. En otras palabras, la tabla debe ser estrictamente bidimensional.

En la Tabla 1, las columnas `car_1`, `car_2` y `car_3` violan este principio al representar múltiples valores de un mismo atributo ("vehículo asignado") en columnas separadas. Para cumplir con la 1FN, debemos "aplanar" la estructura, creando una fila independiente por cada vehículo asignado. El resultado se observa en la **Tabla 2**.

**Tabla 2: Datos Normalizados a la Primera Forma Normal**
| driver_id | driver_name | manager | manager_ext | car_plate |
| :--- | :--- | :--- | :--- | :--- |
| 1022 | Jordan Lee | Alex Mitchell | 345 | XJ4P2T9 |
| 1022 | Jordan Lee | Alex Mitchell | 345 | B9R3L7Q |
| 1022 | Jordan Lee | Alex Mitchell | 345 | M6V1H8Z |
| 1023 | Casey Morgan | Jamie Carter | 123 | XJ4P2T9 |
| 1023 | Casey Morgan | Jamie Carter | 123 | K2S8N4R |
| 1023 | Casey Morgan | Jamie Carter | 123 | L1K4P8S |

La tabla ahora es bidimensional y cada celda contiene un único valor. Sin embargo, la redundancia de datos sigue siendo evidente.

##### 3.1.2 Aplicación de la Segunda Forma Normal (2FN)

La **Segunda Forma Normal (2FN)** requiere que la tabla esté en 1FN y que todos los atributos no clave dependan *completamente* de la clave primaria. Es importante notar que las claves primarias pueden ser **simples** (compuestas por un solo atributo) o **compuestas** (formadas por dos o más atributos). La 2FN es especialmente relevante cuando la clave primaria es compuesta, pues exige que ningún atributo dependa solo de una parte de dicha clave (dependencia parcial).

Si consideramos que la clave primaria única para la Tabla 2 sería la combinación (`driver_id`, `car_plate`), notamos que atributos como `driver_name`, `manager` y `manager_ext` dependen únicamente de `driver_id`, no del vehículo específico. Esta dependencia parcial genera la redundancia observada: los datos del conductor se repiten por cada auto que maneja.

Para resolverlo, separamos la relación de asignación de los datos maestros del conductor. La **Tabla 3** contiene únicamente las asignaciones, mientras que la **Tabla 4** almacena la información única de cada conductor.

**Tabla 3: Tabla de Asignaciones Vehiculares (2FN)**
| driver_id | car_plate |
| :--- | :--- |
| 1022 | XJ4P2T9 |
| 1022 | B9R3L7Q |
| 1022 | M6V1H8Z |
| 1023 | XJ4P2T9 |
| 1023 | K2S8N4R |
| 1023 | L1K4P8S |

**Tabla 4: Datos Maestros de Conductores (2FN)**
| driver_id | driver_name | manager | manager_ext |
| :--- | :--- | :--- | :--- |
| 1022 | Jordan Lee | Alex Mitchell | 345 |
| 1023 | Casey Morgan | Jamie Carter | 123 |

##### 3.1.3 Aplicación de la Tercera Forma Normal (3FN)

La **Tercera Forma Normal (3FN)** exige que la tabla esté en 2FN y que ningún atributo no clave dependa transitivamente de la clave primaria. 

¿Qué significa una **dependencia transitiva**? Ocurre cuando un atributo no clave ($B$) depende de otro atributo no clave ($A$), el cual a su vez depende de la clave primaria ($PK$). La cadena lógica es: $PK \rightarrow A \rightarrow B$. En este escenario, $B$ depende de la clave primaria indirectamente, a través de $A$. La regla de la 3FN busca romper esta cadena haciendo que los atributos no clave dependan directamente y exclusivamente de la clave primaria ("de la clave, de toda la clave y nada más que de la clave").

En la Tabla 4, aunque `driver_id` es la clave primaria simple, existen atributos (`manager`, `manager_ext`) que describen al gerente, no al conductor. Existe una dependencia transitiva: `driver_id` determina al `manager` (quién es el jefe), y el `manager` determina su `manager_ext` (cuál es su extensión telefónica). Para cumplir con la 3FN, debemos extraer los datos de los gerentes a su propia entidad.

La **Tabla 5** muestra la nueva tabla de gerentes, y la **Tabla 6** muestra la tabla de conductores actualizada, que ahora solo hace referencia al gerente mediante su identificador.

**Tabla 5: Datos Maestros de Gerentes (3FN)**
| manager_id | manager_name | manager_ext |
| :--- | :--- | :--- |
| 34 | Alex Mitchell | 345 |
| 35 | Jamie Carter | 123 |

**Tabla 6: Datos Maestros de Conductores Final (3FN)**
| driver_id | driver_name | manager_id |
| :--- | :--- | :--- |
| 1022 | Jordan Lee | 34 |
| 1023 | Casey Morgan | 35 |

##### 3.1.4 Resultado del Modelo Relacional

Tras aplicar las tres primeras formas normales, obtenemos el Diagrama Entidad-Relación (ERD) presentado en la **Figura 1**, el cual elimina la redundancia y las anomalías de actualización básicas.

*Figura 1: Diagrama Entidad-Relación del Modelo Normalizado*

Como actividad de consolidación de conocimiento, se solicita al lector dibujar la **Figura 1: Diagrama Entidad-Relación del Modelo Normalizado** con Enterprise Architect y publíquela en el grupo de Telegram de la UEA. Este ejercicio práctico permitirá verificar la comprensión de las relaciones uno-a-muchos y muchos-a-muchos establecidas durante el proceso de normalización, así como la correcta identificación de las claves primarias y foráneas en el esquema final.

Aunque podríamos normalizar aún más creando una tabla maestra para los vehículos, nos detenemos aquí para mantener el enfoque pedagógico en las relaciones principales. Los beneficios de este diseño incluyen:
*   Eliminación de redundancia de datos.  
*   Integridad referencial reforzada.  
*   Consultas más eficientes.  
*   Mantenimiento simplificado.  

##### 3.1.5. Formas Normales Superiores

Las formas 1FN, 2FN y 3FN cubren la mayoría de los escenarios prácticos. Sin embargo, existen formas superiores como la Forma Normal de Boyce-Codd (BCNF), la 4FN y la 5FN. A continuación, exploramos la BCNF para abordar un caso de borde que la 3FN no resuelve completamente.

#### 3.2 Análisis Avanzado: Forma Normal de Boyce-Codd (BCNF)

La **Forma Normal de Boyce-Codd (BCNF)** es una versión estricta de la 3FN. Su regla establece que *para cada dependencia funcional no trivial $X \rightarrow Y$, $X$ debe ser una superclave*. 

Para clarificar esta notación:
*   $X \rightarrow Y$ significa que el atributo (o conjunto de atributos) $X$ determina funcionalmente a $Y$. Es decir, si conocemos el valor de $X$, podemos determinar unívocamente el valor de $Y$.  
*   Una **superclave** es cualquier conjunto de atributos que identifica de manera única una fila en la tabla.  
*   Por lo tanto, la regla exige que **cualquier atributo que sirva para determinar otro atributo debe ser, por sí mismo, capaz de identificar de forma única la fila completa**. Si un determinante no es una superclave, la tabla viola la BCNF.  

**Escenario: Gestión de Capacitaciones**

Supongamos que la empresa registra la capacitación de los conductores bajo las siguientes reglas:
1.  Un conductor toma varios cursos.  
2.  Cada curso tiene un único instructor especializado.  
3.  Cada instructor enseña solo un tipo de curso.  

**Tabla 7: Registro de Capacitación (Cumple 3FN, viola BCNF)**
| driver_id | course_name | instructor_name |
| :--- | :--- | :--- |
| 1022 | Defensive Driving | Sarah Jenkins |
| 1022 | Hazardous Materials | David Ross |
| 1023 | Defensive Driving | Sarah Jenkins |
| 1024 | Hazardous Materials | David Ross |

**Análisis de Dependencias:**

Las claves candidatas compuestas son (`driver_id`, `course_name`) y (`driver_id`, `instructor_name`).
Sin embargo, existe la dependencia `course_name` $\rightarrow$ `instructor_name` (el curso determina quién lo imparte). Como `course_name` por sí solo no es una superclave (varios conductores toman el mismo curso, por lo que `course_name` no identifica una fila única), la tabla viola la BCNF. Esto genera una anomalía de actualización: si cambiamos el instructor de un curso, debemos actualizar múltiples filas.

**Solución BCNF:**

Descomponemos la tabla en dos: una para la relación muchos-a-muchos y otra para la especialización del instructor.

**Tabla 8: Asignación de Cursos (BCNF)**
| driver_id (FK) | course_name (FK) |
| :--- | :--- |
| 1022 | Defensive Driving |
| 1022 | Hazardous Materials |
| ... | ... |

**Tabla 9: Especialización de Instructores (BCNF)**
| course_name (PK) | instructor_name |
| :--- | :--- |
| Defensive Driving | Sarah Jenkins |
| Hazardous Materials | David Ross |

Con esta estructura, la información del instructor se almacena una sola vez, garantizando la consistencia. La interconexión lógica de estas nuevas entidades se visualiza en la **Figura 2**, donde se aprecia cómo la tabla de especializaciones actúa como el origen de verdad para la relación entre cursos e instructores, eliminando la redundancia previa.

*Figura 2: Diagrama Entidad-Relación del Modelo Normalizado a BCNF*

Se solicita al lector dibujar la **Figura 2: Diagrama Entidad-Relación del Modelo Normalizado a BCNF** con Enterprise Architect. Publíquela en el grupo de Telegram de la UEA.

#### 3.3 Diccionario de Datos

El Diccionario de Datos documenta los metadatos del esquema, definiendo tipos de datos, restricciones y significados lógicos. Es la guía técnica para la implementación física.

**Tabla 10: Diccionario – Tabla `managers`**
| Campo | Tipo | Restricción | Descripción |
| :--- | :--- | :--- | :--- |
| `manager_id` | INT | PK, NOT NULL | ID único del gerente. |
| `manager_name` | VARCHAR(100) | NOT NULL | Nombre completo. |
| `manager_ext` | VARCHAR(10) | NULLABLE | Extensión telefónica. |

**Tabla 11: Diccionario – Tabla `drivers`**
| Campo | Tipo | Restricción | Descripción |
| :--- | :--- | :--- | :--- |
| `driver_id` | INT | PK, NOT NULL | ID único del conductor. |
| `driver_name` | VARCHAR(100) | NOT NULL | Nombre completo. |
| `manager_id` | INT | FK, NULLABLE | Hace referencia a `managers.manager_id`. |

**Tabla 12: Diccionario – Tabla `driver_car_assignments`**
| Campo | Tipo | Restricción | Descripción |
| :--- | :--- | :--- | :--- |
| `driver_id` | INT | PK, FK, NOT NULL | Hace referencia a conductor. |
| `car_plate` | VARCHAR(10) | PK, NOT NULL | Placa del vehículo. |

#### 3.4 Implementación SQL: De la Teoría a la Práctica

Finalmente, traducimos el modelo lógico a instrucciones SQL. Esto materializa las reglas de integridad descubiertas durante la normalización.

```sql
-- Tabla de Gerentes (Entidad Independiente)
CREATE TABLE managers (
    manager_id INT PRIMARY KEY,
    manager_name VARCHAR(100) NOT NULL,
    manager_ext VARCHAR(10)
);

-- Tabla de Conductores (Hace referencia a Gerentes)
CREATE TABLE drivers (
    driver_id INT PRIMARY KEY,
    driver_name VARCHAR(100) NOT NULL,
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES managers(manager_id)
);

-- Tabla de Asignaciones (Relación Muchos-a-Muchos)
CREATE TABLE driver_car_assignments (
    driver_id INT,
    car_plate VARCHAR(10),
    PRIMARY KEY (driver_id, car_plate),
    FOREIGN KEY (driver_id) REFERENCES drivers(driver_id)
);
```

Para el caso BCNF de capacitaciones:

```sql
-- Especialización de Instructores
CREATE TABLE instructor_specializations (
    course_name VARCHAR(100) PRIMARY KEY,
    instructor_name VARCHAR(100) NOT NULL
);

-- Registro de Capacitación
CREATE TABLE driver_training_records (
    driver_id INT,
    course_name VARCHAR(100),
    PRIMARY KEY (driver_id, course_name),
    FOREIGN KEY (driver_id) REFERENCES drivers(driver_id),
    FOREIGN KEY (course_name) REFERENCES instructor_specializations(course_name)
);
```

Al codificar estas estructuras, estamos integrando las reglas de negocio directamente en la base de datos, previniendo inconsistencias antes de que ocurran.

Para consolidar los conocimientos adquiridos, se invita al lector a materializar este diseño creando una base de datos real en **PostgreSQL**. Se sugiere nombrar la base de datos como `fleet_management_db` para mantener la consistencia con el dominio del ejercicio.

Como evidencia de la correcta implantación del esquema y como mecanismo de validación colaborativa, se solicita realizar un respaldo completo (*backup*) de la base de datos utilizando la utilidad `pg_dump`. El archivo SQL resultante deberá ser publicado en el grupo de Telegram del curso. Esta práctica no solo verifica la sintaxis de las instrucciones `CREATE TABLE`, sino que también asegura que las restricciones de integridad referencial (Claves Foráneas) hayan sido correctamente establecidas por el sistema gestor. 

#### 3.5. Implementación

Para consolidar los conocimientos adquiridos, se invita al lector a materializar este diseño creando una base de datos real en **PostgreSQL**. Se sugiere nombrar la base de datos como `fleet_management_db` para mantener la consistencia con el dominio del ejercicio.

Como evidencia de la correcta implantación del esquema y como mecanismo de validación colaborativa, se solicita realizar un respaldo completo (*backup*) de la base de datos utilizando la utilidad `pg_dump`. El archivo SQL resultante deberá ser publicado en el grupo de Telegram del curso. Esta práctica no solo verifica la sintaxis de las instrucciones `CREATE TABLE`, sino que también asegura que las restricciones de integridad referencial (Claves Foráneas) hayan sido correctamente establecidas por el sistema gestor.

Para generar el respaldo, ejecute el siguiente comando en su terminal:

```bash
pg_dump -U [su_usuario] -d fleet_management_db > fleet_management_backup.sql
```

*(Sustituya `[su_usuario]` por el nombre de usuario de PostgreSQL que esté utilizando, comúnmente `postgres` o su usuario de sistema).*

Adicionalmente, es fundamental documentar el proceso técnico mediante los registros del sistema (*logs*). Para ello, incluya en su entrega los *logs* de la máquina virtual donde realizó el ejercicio. Si está trabajando en un entorno **Fedora 44**, puede recuperar los registros relevantes de la sesión actual o del servicio de PostgreSQL utilizando el siguiente comando en la terminal:

```bash
# Para ver los logs del sistema desde el último arranque
journalctl -b

# O específicamente para filtrar mensajes relacionados con PostgreSQL
journalctl -u postgresql.service --since "today"
```

Guarde la salida de estos comandos en un archivo de texto (por ejemplo, `evidencia_logs_[matricula].txt`) y adjúntelo junto con el *dump* de la base de datos (`fleet_management_backup_[matricula].sql`). Esto permitirá verificar la secuencia de operaciones realizadas y garantizará la trazabilidad del ejercicio práctico.

#### 4. Conclusiones

Este ejercicio ha demostrado cómo la normalización transforma datos desestructurados en un modelo relacional robusto. Desde la eliminación de grupos repetitivos en la 1FN hasta la resolución de dependencias transitivas en la 3FN y determinantes no clave en la BCNF, cada paso contribuye a la integridad y eficiencia del sistema. La documentación mediante diccionarios de datos y su posterior implementación en SQL cierran el ciclo de desarrollo, asegurando que el diseño teórico se traduzca en una solución técnica fiable y escalable.

#### 5.Referencia

Koseoglu, K. (2025). *SQL: The practical guide*. Rheinwerk Publishing.
