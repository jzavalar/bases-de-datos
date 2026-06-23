### Laboratorio 11: Proyecto de Mitad de Curso – Análisis y Diseño de la Base de Datos para Maven Movies
### Evaluación parcial 1

**Dr. Jesús Zavala Ruiz**  
*22 de junio de 2026*

---

#### 1. Introducción

Diseñar una base de datos eficiente va mucho más allá de organizar tablas; implica construir los cimientos de sistemas íntegros, eficientes y escalables. Un diseño descuidado no solo genera anomalías de inserción, actualización y eliminación, sino que también introduce una redundancia que compromete la confiabilidad de toda la información. En este sentido, el análisis de datos y el diseño de bases de datos forman un ciclo continuo donde la extracción de información valida constantemente la calidad del modelo subyacente.

Este laboratorio integra los conceptos de normalización estudiados en el Laboratorio 10 con la implementación técnica en un entorno de producción real. Partiremos de un archivo de datos desestructurado que simula las operaciones heredadas de la compañía **Maven Movies**, y lo guiaremos a través de la preparación de la infraestructura, la resolución de requerimientos analíticos complejos y la documentación formal del esquema hasta la Tercera Forma Normal (3NF) y la Forma Normal de Boyce-Codd (BCNF). 

A diferencia de los ejercicios puramente teóricos, aquí deberá entregar artefactos tangibles: un modelo lógico normalizado, un diccionario de datos exhaustivo, un modelo físico ejecutable y un script de respaldo automatizado. De esta manera, cerraremos el ciclo completo del desarrollo de bases de datos relacionales, desde el análisis conceptual hasta la validación operativa.

#### 2. Fundamentos del Caso de Estudio

La compañía **Maven Movies** se encuentra en proceso de renovar su póliza de seguros corporativos. Sin embargo, el suscriptor principal (*Lead Underwriter*), Joe Scardycat, ha emitido una comunicación oficial señalando que la información comercial almacenada en sus sistemas no se ha actualizado en varios años. Sin datos fidedignos, la aseguradora no puede evaluar los riesgos ni aprobar la renovación.

##### 2.1 La Carta del Suscriptor

> **Estimada Gerencia de Maven Movies,**
>
> En nuestra revisión de su solicitud de renovación de póliza, hemos notado que la información de su negocio no ha sido actualizada en varios años. Para poder evaluar con precisión el riesgo y aprobar la renovación de su póliza, necesitaremos que nos proporcione toda la siguiente información.
>
> Atentamente,  
> **Joe Scardycat, Suscriptor Principal**

Esta carta define el contexto de negocio: no estamos escribiendo consultas SQL por simple práctica académica, sino respondiendo a una necesidad legal y financiera urgente. Cada dato extraído tiene implicaciones directas en la continuidad operativa de la empresa, haciendo que la precisión de la información sea crítica para la toma de decisiones y la evaluación de riesgos.

#### 3. Insumo de Datos

El punto de partida para el diseño de la base de datos es el archivo [`Lab_11.2_reporte_maestro_maven.csv`](https://github.com/jzavalar/bases-de-datos/blob/main/Lab_11.2_reporte_maestro_maven.csv), disponible en los recursos del curso. Este archivo contiene **exactamente 500 registros** que simulan la "verdad cruda" de operaciones heredadas, reflejando la estructura típica de una hoja de cálculo sin ningún tipo de normalización.

La estructura del archivo se define mediante el siguiente encabezado:

```text
ID_Renta,Fecha_Renta,Cliente_Nombre,Cliente_Email,Cliente_Activo,Tienda_ID,Tienda_Direccion,Staff_Nombre,Staff_Email,Pelicula_Titulo,Pelicula_Categoria,Pelicula_Costo_Reemplazo,Inventario_ID,Pago_Monto,Pago_Fecha_Pago
```

Al analizar este reporte, identificará varias deficiencias estructurales que deberá corregir mediante el proceso de normalización:  
-   **Valores compuestos:** Campos que contienen múltiples valores atómicos en una sola celda.  
-   **Redundancia masiva:** Datos repetidos sistemáticamente en múltiples filas.  
-   **Dependencias transitivas:** Atributos no clave que dependen de otros atributos no clave.  
-   **Determinantes no superclave:** Atributos que determinan funcionalmente a otros sin ser superclaves.  

**Advertencia crítica:** No modifique ni limpie este archivo manualmente. Su propósito es servir como la fuente única de verdad para validar y hacer evidente cómo el proceso de normalización elimina todas las anomalías presentes. El análisis sistemático de este reporte le permitirá comprobar que cada paso de normalización responde a una anomalía concreta y medible, y no a una abstracción teórica.

#### 4. Infraestructura y Entorno Técnico

Para garantizar la reproducibilidad y el uso de estándares abiertos, este laboratorio abandona los entornos propietarios en favor de soluciones robustas y libres, alineadas con las mejores prácticas de la industria.

##### 4.1 Servidor Web: Apache HTTP Server

Utilizaremos [Apache HTTP Server](https://httpd.apache.org/), un servidor web de software libre que ofrece alta estabilidad, seguridad y es ampliamente adoptado en la industria. Su inclusión en el laboratorio permitirá, en fases posteriores, exponer servicios de visualización de datos, APIs o interfaces de administración para la base de datos. Es fundamental instalar su versión estable más reciente y configurar adecuadamente los servicios del sistema y el firewall para garantizar la disponibilidad y seguridad del entorno.

##### 4.2 Motor de Base de Datos: MariaDB

Utilizaremos [MariaDB](https://mariadb.org/), un sistema gestor de bases de datos relacional de software libre que ofrece compatibilidad total con MySQL y características avanzadas de optimización. Es fundamental instalar su versión estable más reciente y configurar adecuadamente las variables de entorno para garantizar el rendimiento y la seguridad del sistema.

##### 4.3 Herramienta de Gestión: DBeaver Community Edition

Como herramienta de gestión, emplearemos [DBeaver Community Edition](https://dbeaver.io/). Esta herramienta universal permite visualizar esquemas, editar datos y generar diagramas Entidad-Relación (ER) de manera intuitiva, consolidándose como el estándar para el analista moderno por su capacidad para conectarse a múltiples motores de bases de datos.

##### 4.4 Configuración del Entorno

Asegúrese de verificar que el servicio de MariaDB esté activo y que DBeaver pueda establecer una conexión exitosa a dos bases de datos específicas:  
-   **`maven_movies_reporte`:** Entorno aislado para cargar el reporte maestro sin normalizar. (Desarrollo). (También podría ser una tabla separada de la base de datos `maven_movies`)   
-   **`maven_movies`:** Base de datos productiva que alojará el diseño normalizado. (Producción)

Esta separación de entornos previene la contaminación de datos, permite experimentar sin riesgos y refleja las prácticas profesionales de segregación entre desarrollo y producción.

#### 5. Requerimientos Analíticos del Suscriptor

El objetivo del proyecto es aprovechar sus habilidades SQL para extraer los datos y poblar la base de datos normalizada. Una regla fundamental de este ejercicio es que **cada pregunta puede responderse consultando una sola tabla**. Esto refuerza la comprensión de la estructura interna de cada entidad y evita dependencias prematuras antes de validar la integridad individual de las tablas. 

El verdadero desafío analítico no reside solo en escribir la sintaxis SQL, sino en mapear correctamente la pregunta de negocio con la entidad técnica adecuada; la sintaxis es solo el vehículo, pero el conocimiento del dominio es el motor.

A continuación, se detallan los ocho requerimientos derivados de la carta del suscriptor:

##### 5.1 Listado de Personal

Se requiere identificar a todo el capital humano asignado a las operaciones.  
-   **Requerimiento:** Lista de miembros del personal incluyendo nombre, apellido, correo electrónico e identificador de tienda (`store_id`).  
-   **Entidad Objetivo:** `staff`  

##### 5.2 Inventario por Ubicación

La distribución física de activos (inventario) es clave para la evaluación de riesgos patrimoniales.  
-   **Requerimiento:** Conteo separado de artículos de inventario para cada una de las dos tiendas.  
-   **Entidad Objetivo:** `inventory`  

##### 5.3 Base de Clientes Activos 

El valor de la póliza depende directamente del volumen de usuarios (clientes) operativos.  
-   **Requerimiento:** Conteo de clientes activos (`active = 1`) desagregado por tienda.  
-   **Entidad Objetivo:** `customer`  

##### 5.4 Evaluación de Riesgo de Datos

En la era digital, la cantidad de Información de Identificación Personal (PII) determina la exposición ante brechas de seguridad.
-   **Requerimiento:** Conteo total de direcciones de correo electrónico de clientes almacenadas.  
-   **Entidad Objetivo:** `customer`  

##### 5.5 Diversidad del Catálogo

La retención de clientes está correlacionada con la variedad de la oferta.  
-   **Requerimiento:**  
    1.  Conteo de títulos únicos de películas (film) en inventario por tienda (inventory).  
    2.  Conteo de categorías (category) únicas de filmes disponibles.  
-   **Entidades Objetivo:** `inventory`, `category` y `film_category`  

##### 5.6 Valoración de Activos

El costo de reposición define el monto asegurado necesario.  
-   **Requerimiento:** Costo mínimo, máximo y promedio de reemplazo de las películas.  
-   **Entidad Objetivo:** `film`  

##### 5.7 Monitoreo Transaccional

Para prevenir fraudes internos, se deben establecer líneas base de comportamiento financiero.  
-   **Requerimiento:** Promedio y monto máximo de pagos procesados (payment) históricamente.  
-   **Entidad Objetivo:** `payment`  

##### 5.8 Perfilamiento de Clientes

Identificar a los clientes de alto valor permite segmentar estrategias de retención.  
-   **Requerimiento:** Lista de IDs de clientes ordenada descendientemente por el conteo total de alquileres realizados (rental).  
-   **Entidad Objetivo:** `rental`  

#### 6. Entregables Formales del Proyecto

Más allá de la ejecución de consultas, este laboratorio exige materializar el conocimiento en documentos técnicos estandarizados. Estos entregables constituyen la evidencia de su competencia profesional y deben seguir los lineamientos presentados en el Laboratorio 10.

##### 6.1 Modelo Lógico Normalizado a 3NF
Deberá presentar un Diagrama Entidad-Relación (ERD) que refleje la estructura óptima de la base de datos, validando dos aspectos fundamentales. Primero, la **normalización**: todas las tablas deben cumplir, por lo menos, con la Tercera Forma Normal (3NF), eliminando dependencias transitivas y redundancias, documentando cómo aplicó las formas 1FN, 2FN, 3FN y, donde aplique, BCNF. Segundo, las **vistas de reporte**: el modelo debe incluir la definición de *views* diseñadas para encapsular la lógica de los reportes solicitados, actuando como una capa de abstracción que protege la integridad de las tablas base y simplifica el acceso a la información recurrente.

##### 6.2 Diccionario de Datos
Siguiendo el estándar del Laboratorio 10, documentará exhaustivamente el esquema. Deberá detallar el contenido (tablas, columnas, tipos de datos, restricciones, índices y descripciones semánticas) y explicar su propósito en el contexto del negocio de Maven Movies, vinculando cada decisión de diseño con los requerimientos analíticos del suscriptor.

##### 6.3 Modelo Físico
Traducirá el modelo lógico a código ejecutable mediante un script DDL completo compatible con MariaDB. Debe incluir sentencias `CREATE TABLE`, claves foráneas, índices y las vistas mencionadas, siendo capaz de recrear la base de datos desde cero en un entorno limpio e integrando las reglas de negocio directamente en la base de datos.

##### 6.4 Script de Respaldo Automatizado
La disponibilidad de los datos es tan importante como su integridad. Desarrollará un script que utilice herramientas nativas (`mysqldump` o DBeaver) para generar copias de seguridad completas. Como evidencia de la correcta implantación, deberá realizar un respaldo completo y documentar el proceso mediante los registros del sistema (*logs*) de la máquina virtual, filtrados para mostrar las operaciones relevantes de MariaDB. Esta práctica asegurará que las restricciones de integridad referencial hayan sido correctamente establecidas por el sistema gestor.

#### 7. Conclusiones

Este proyecto sintetiza la teoría de la normalización con la práctica del análisis de datos empresarial. Al transitar desde la interpretación de una carta de negocios hasta la generación de un script de respaldo en MariaDB, validará no solo su capacidad técnica para escribir SQL, sino su comprensión holística del ciclo de vida de los datos. 

Desde la eliminación de grupos repetitivos en la 1FN hasta la resolución de dependencias transitivas en la 3FN y determinantes no clave en la BCNF, cada paso contribuye a la integridad y eficiencia del sistema. La documentación mediante diccionarios de datos y su posterior implementación en SQL cierran el ciclo de desarrollo, asegurando que el diseño teórico se traduzca en una solución técnica fiable y escalable.

#### 8. Referencias

-   Koseoglu, K. (2025). *SQL: The practical guide*. Rheinwerk Publishing.
-   MariaDB Foundation. (2026). *MariaDB Server Documentation*. https://mariadb.org/
-   DBeaver Corp. (2026). *DBeaver Community Edition*. https://dbeaver.io/

---

### Laboratorio 11 – Análisis y Diseño para Maven Movies
### Guía Híbrida de Ejecución
### Enfoque: Asistido por IA con Validación Humana
**Tiempo estimado: 90 minutos**

**Dr. Jesús Zavala Ruiz**  
*Junio de 2026*

---

Bienvenido a la guía de ejecución del Laboratorio 11. Dado que ya has completado el Laboratorio 10, los conceptos fundamentales de normalización (desde la 1FN hasta la BCNF) ya forman parte de tu repertorio técnico. El objetivo de esta sesión de 90 minutos no es repasar la teoría, sino ejecutar un flujo de trabajo profesional e híbrido. Utilizarás la Inteligencia Artificial como copiloto para acelerar la sintaxis y la generación de scripts, mientras mantienes el control crítico sobre la lógica de negocio, la integridad de los datos, el rendimiento y la seguridad del entorno.

#### Principios de Interacción con la IA

Para aprovechar al máximo esta herramienta y evitar respuestas genéricas o erróneas, es fundamental establecer reglas claras de interacción desde el inicio.

**1. La Regla de Oro: La IA Siempre Explica**

En este laboratorio, **nunca aceptes una respuesta de la IA sin una justificación**. Cada vez que solicites código, comandos o decisiones de diseño, debes exigir que la herramienta explique su razonamiento. Esto te permitirá validar críticamente la solución, comprender el *porqué* detrás de cada decisión técnica, detectar posibles alucinaciones y documentar tu proceso de toma de decisiones. Si la IA te entrega un script sin explicar por qué utiliza ciertas funciones, tablas o tipos de datos, pídele que lo justifique antes de ejecutarlo.

**2. La Fórmula del Prompt Efectivo: Contexto, Rol y Tarea**

Para que un chatbot de IA arroje respuestas de alta calidad, no basta con hacer preguntas aisladas. Debes estructurar tus *prompts* integrando tres elementos fundamentales:  

-   **Contexto:** Describe el entorno técnico y el estado actual del proyecto (ej. *"Estoy trabajando en Fedora 44 con MariaDB, partiendo de un CSV de 500 registros en una base de datos llamada `maven_movies_reporte`..."*).  
-   **Rol:** Asígnale una identidad experta para elevar el nivel de la respuesta (ej. *"Actúa como un Arquitecto de Datos Senior y experto en optimización de MariaDB..."*).  
-   **Tarea:** Especifica con precisión qué necesitas que haga (ej. *"Escribe las consultas `INSERT INTO ... SELECT ...` para poblar las tablas maestras..."*).  

A partir del Bloque 2, serás tú quien redacte los *prompts* aplicando esta fórmula, sin olvidar la regla de oro de exigir siempre la explicación. A lo largo de la guía, encontrarás **Puntos de Control (Checkpoints)**; no avances al siguiente bloque hasta haber verificado el punto actual.

#### Bloque 1: Infraestructura y Entorno (15 min)

El primer paso es levantar la pila tecnológica que sostendrá el proyecto. Trabajaremos con MariaDB como motor relacional, un servidor web para posibles visualizaciones futuras y DBeaver como interfaz gráfica. Además, es fundamental garantizar un acceso seguro y remoto mediante el uso de criptografía de clave pública (PKI) y túneles SSH, evitando la exposición directa de los puertos de la base de datos.

**Acciones:**  

1.  **Servidor Web:** Instala un servidor web (por ejemplo, Apache `httpd` o Nginx) y asegúrate de que el servicio esté activo y habilitado.  
2.  **MariaDB:** Instala el servidor y cliente. Ejecuta el script de seguridad inicial (`mysql_secure_installation`). No delegues la toma de decisiones de seguridad a la IA; es tu primera línea de defensa.  
3.  **DBeaver:** Descarga e instala DBeaver Community en tu máquina local.  
4.  **Configuración de Acceso Seguro (SSH y PKI):**  
    -   Genera un par de llaves SSH (pública y privada) en tu máquina local si aún no cuentas con ellas.  
    -   Copia tu llave pública al usuario de la máquina virtual para habilitar el acceso seguro sin el uso de contraseñas.  
    -   Configura un túnel SSH para redirigir el puerto de MariaDB (3306) de la máquina virtual a tu `localhost`.  

**Prompt Inicial (Ejemplo de Interacción):**

> [Rol] Actúa como un Administrador de Sistemas Linux experto en seguridad. <br> [Contexto] Estoy configurando un entorno de desarrollo en Fedora 44 para un proyecto de bases de datos. <br> [Tarea] Dame los comandos exactos para instalar MariaDB y Apache, iniciar sus servicios, habilitarlos en el arranque y verificar su estado. <br> [Regla] Explica qué hace cada comando y por qué es necesario para la seguridad y estabilidad del entorno."

**Punto de Control 1:**  

- [ ] Sube al chat de Telegram con tu profesor cada uno de los entregables conforme progreses.  
- [ ] Ejecuta `systemctl status httpd` y `systemctl status mariadb` en la máquina virtual. Ambos deben aparecer como "active (running)".  
- [ ] Confirma que puedes acceder a la máquina virtual desde tu terminal local por contraseña y utilizando tu llave privada (*SSH* con *PKI*).  
- [ ] Establece un túnel *SSH* exitoso para redirigir el puerto 3306 de la máquina virtual a tu `localhost`.  
- [ ] En *DBeaver*, configura la conexión para que utilice el túnel SSH y tu llave privada, realizando un "Test Connection" exitoso a `localhost`.  
- [ ] Toma una captura de pantalla de los estados de los servicios y de la conexión exitosa en *DBeaver* y guárdala como `evidencia_01_infraestructura.png`.  
  

#### Bloque 2: Aislamiento del Reporte Maestro (10 min)

Necesitamos un entorno seguro y aislado para analizar los 500 registros del archivo `lab_11.2_reporte_maestro_maven.csv`, evitando así contaminar la base de datos final. Además, por principios de seguridad, no trabajaremos directamente con el usuario `root` en las bases de datos de aplicación.

**Acciones:**  

1.  **Crear Usuario Alumno:** Genera un usuario llamado `alumno` con una contraseña robusta y otórgale los privilegios necesarios.  
2.  **Base de Datos del Reporte:** Crea la base de datos `maven_movies_reporte` con codificación `utf8mb4`.  
3.  **Carga del CSV:** Crea la tabla `reporte_crudo` con las 15 columnas del encabezado y carga el archivo CSV utilizando `LOAD DATA INFILE`.  

**Formulación del Prompt por el Estudiante:**

Redacta tu propio *prompt* aplicando la fórmula (Contexto, Rol, Tarea) solicitando a la IA el script *SQL* para la creación del usuario, la base de datos, la tabla y la carga del CSV. **Recuerda exigirle que explique la elección de cada tipo de dato y las consideraciones de seguridad aplicadas.**

**Punto de Control 2:**

- [ ] Sube al chat de Telegram con tu profesor cada uno de los entregables conforme progreses.  
- [ ] Inicia sesión en *MariaDB* con el usuario `alumno`.  
- [ ] Ejecuta `SELECT COUNT(*) FROM maven_movies_reporte.reporte_crudo;`. El resultado debe ser exactamente **500**.  
- [ ] Guarda la salida en `evidencia_02_carga_reporte.txt`.  

#### Bloque 3: Análisis, Normalización y Carga de Datos (30 min)

Aquí es donde tu criterio humano cobra mayor relevancia. La IA puede sugerir estructuras, pero tú debes validar que las dependencias funcionales del reporte maestro se resuelvan correctamente en tu nuevo esquema 3NF/BCNF.

**Acciones:**

1.  **Diseño del Esquema:** Analiza las anomalías del reporte (valores compuestos en nombres, dependencias transitivas de direcciones y correos). Diseña las tablas normalizadas.  
2.  **Crear BD Normalizada:** Crea la base de datos `maven_movies`.  
3.  **Extraer, Transformar y Cargar (ETL):** Escribe las consultas `INSERT INTO ... SELECT ...` para extraer los datos únicos desde el reporte crudo y poblar las nuevas tablas normalizadas.  

**Formulación del Prompt por el Estudiante:**

Solicita a la IA el esquema normalizado y los scripts ETL usando la fórmula. **Exige que la IA detalle paso a paso cómo aplicó cada forma normal (1FN, 2FN, 3FN, BCNF), qué anomalías específicas eliminó y por qué utiliza ciertas cláusulas (como `DISTINCT` o `GROUP BY`) en los scripts de inserción.**

**Punto de Control 3:**

- [ ] Sube al chat de Telegram con tu profesor cada uno de los entregables conforme progreses.  
- [ ] Verifica que las tablas en `maven_movies` tengan datos (ej. `SELECT COUNT(*) FROM maven_movies.staff;`).  
- [ ] Asegúrate de que no haya datos duplicados en las tablas maestras.  
- [ ] Guarda el script de inserción (ETL) como `etl_normalizacion.sql`.  

#### Bloque 4: Vistas, Optimización y Cumplimiento de Requerimientos (20 min)

El suscriptor Joe Scardycat exige 8 respuestas específicas. En lugar de obligarlo a escribir consultas complejas, encapsularás la lógica en vistas. Sin embargo, no basta con que funcionen; deben ser eficientes.

**Acciones:**

1.  **Creación de Vistas:** Diseña una vista para cada uno de los 8 requerimientos del suscriptor.  
2.  **Análisis de Rendimiento (`EXPLAIN`) e Índices:** Por cada vista creada, debes ejecutar `EXPLAIN SELECT ...`. Analiza el plan de ejecución. Si detectas escaneos completos de tabla (*Full Table Scan* o tipo `ALL`), debes explorar el uso de índices.  
3.  **Prueba Uno a Uno:** Verifica que los resultados coincidan con lo que el suscriptor pidió.  

**Formulación del Prompt por el Estudiante (Enfoque en Optimización):**

Pide a la IA las sentencias `CREATE VIEW`. Una vez generadas y ejecutado el `EXPLAIN`, formula un nuevo prompt pegando la salida del `EXPLAIN` y pide a la IA que proponga la creación de índices específicos para mejorar el desempeño, exigiendo que explique cómo el índice altera el plan de ejecución y por qué mejora la lectura.

**Punto de Control 4:**  

- [ ] Sube al chat de Telegram con tu profesor cada uno de los entregables conforme progreses.  
- [ ] Ejecuta las 8 vistas y confirma que devuelven resultados coherentes.  
- [ ] Ejecuta `EXPLAIN` en al menos dos vistas complejas. Si fue necesario, crea los índices propuestos y vuelve a ejecutar `EXPLAIN` para confirmar la mejora.  
- [ ] Guarda las definiciones de las vistas y los índices en `vistas_e_indices.sql`.  

#### Bloque 5: Respaldo, Logs y Cierre (15 min)

La disponibilidad de los datos es tan crítica como su integridad. Cerraremos el laboratorio asegurando la base de datos y dejando trazabilidad de tu sesión de trabajo.

**Acciones:**

1.  **Respaldo:** Utiliza `mysqldump` para generar un archivo `.sql` comprimido de la base de datos `maven_movies`.  
2.  **Recuperación de Logs:** Extrae los registros del sistema (`journalctl`) de MariaDB correspondientes a tu sesión de trabajo.  

**Formulación del Prompt por el Estudiante:**

Solicita los comandos de terminal para el respaldo comprimido con fecha y la extracción de logs. Exige a la IA que explique qué hace cada *flag* del comando `mysqldump`, por qué es importante comprimir el backup y cómo interpretar los logs que estamos extrayendo para auditar la sesión.

**Punto de Control 5:**

- [ ] Sube al chat de Telegram con tu profesor cada uno de los entregables conforme progreses.  
- [ ] Verifica que el archivo de respaldo existe y tiene un tamaño mayor a 0 KB.  
- [ ] Guarda los logs filtrados en `evidencia_05_logs_sesion.txt`.  
- [ ] (Opcional pero recomendado) Intenta restaurar el respaldo en una base de datos de prueba para garantizar que no hay corrupción.  

#### Cierre del Laboratorio

Has completado el ciclo de vida del proyecto. Partiste de un archivo plano con 500 registros redundantes, pasaste por un proceso de normalización asistido por IA, creaste un esquema relacional robusto, optimizaste las consultas mediante el análisis de `EXPLAIN` y la creación de índices, y aseguraste la continuidad operativa con un respaldo. 

La IA aceleró tu sintaxis, pero tu capacidad de redactar *prompts* precisos (usando Contexto, Rol y Tarea), exigir explicaciones, analizar el rendimiento y validar críticamente cada respuesta fue lo que transformó este ejercicio en un verdadero trabajo de ingeniería de software. En el mundo profesional, no basta con que el código funcione; debe ser eficiente, seguro y, sobre todo, debes entender por qué lo es. ¡Excelente trabajo!

---

### Rúbrica de Evaluación: Laboratorio 11 – Análisis y Diseño para Maven Movies
**Ponderación Total:** 100%

| Criterio de Evaluación | Ponderación | Excelente (100-90%) | Satisfactorio (89-75%) | Insuficiente (74-60%) | Deficiente (<60%) |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **1. Infraestructura, Seguridad y Acceso (PKI/SSH)** | 10% | MariaDB y Apache activos. Configura acceso SSH con PKI (llave pública/privada). Túnel SSH funcional para DBeaver. Evidencias claras y seguras. | Servicios activos. El acceso por PKI o túnel SSH funciona, pero falta documentar uno de los pasos o la configuración presenta redundancias menores. | Servicios instalados pero no activos, o el acceso se realiza únicamente por contraseña, omitiendo el uso de PKI y túnel SSH. | No instala los servicios o no hay evidencia de conexión remota segura. |
| **2. Gestión del Insumo de Datos (Reporte Maestro)** | 5% | Crea BD `maven_movies_reporte`, usuario `alumno` con privilegios adecuados, y carga exactamente 500 registros en `reporte_crudo` sin alterar el CSV. | Carga los 500 registros correctamente, pero omite la creación del usuario `alumno` o utiliza el usuario `root` para la operación. | La carga de datos es incompleta (menos de 500 registros) o la estructura de la tabla no coincide con el encabezado del CSV. | No realiza la carga del reporte maestro o altera el archivo original. |
| **3. Normalización y Modelo Lógico (3NF/BCNF)** | 20% | ERD en 3NF/BCNF impecable. Elimina dependencias transitivas y redundancias. Incluye vistas de reporte. Documenta rigurosamente la aplicación de las formas normales. | ERD en 3NF, pero con omisiones menores en la documentación teórica de las formas normales o falta definir alguna vista de reporte en el diagrama. | El modelo presenta dependencias transitivas o redundancias evidentes (no cumple 3NF). Falta justificación teórica. | No presenta modelo lógico o este es una copia directa del reporte plano sin normalizar. |
| **4. Procesos ETL e Integridad de Datos** | 15% | Script ETL (`INSERT INTO ... SELECT`) funcional y eficiente. Extrae datos únicos correctamente. No hay duplicados en tablas maestras. Mapeo correcto de FKs. | Script funcional pero con ineficiencias menores (ej. subconsultas innecesarias) o algún dato duplicado aislado que no afecta la integridad global. | El script ETL falla, genera errores de FK, o deja datos redundantes en las tablas maestras por mal uso de `DISTINCT`/`GROUP BY`. | No presenta script ETL o los datos no se cargan dinámicamente desde el reporte crudo. |
| **5. Vistas, Optimización y Desempeño** | 20% | Las 8 vistas funcionan y responden a los requerimientos. Ejecuta `EXPLAIN`, identifica *Full Table Scans* y crea índices justificando técnicamente la mejora de rendimiento. | Las 8 vistas funcionan, pero el análisis de `EXPLAIN` es superficial o no crea índices a pesar de detectar ineficiencias en el plan de ejecución. | Faltan vistas o algunas no devuelven los datos correctos. No hay evidencia de uso de `EXPLAIN` ni optimización. | No presenta vistas ni análisis de rendimiento. |
| **6. Documentación Técnica (Diccionario y DDL)** | 15% | Diccionario de datos exhaustivo y contextualizado al negocio. Script DDL completo, ejecutable en MariaDB, con FKs, índices y vistas. Orden de creación correcto. | Diccionario completo pero con descripciones genéricas. DDL ejecutable pero con omisiones menores (ej. falta un índice o comentarios). | Diccionario incompleto. DDL presenta errores de sintaxis, viola el orden de dependencias o no es ejecutable en un entorno limpio. | No presenta documentación técnica o el DDL es inválido. |
| **7. Continuidad Operativa (Respaldo y Logs)** | 5% | Script de respaldo (`mysqldump`) comprimido funcional. Logs de `journalctl` extraídos, filtrados correctamente y guardados como evidencia. | Respaldo funcional pero sin comprimir, o logs extraídos sin el filtro adecuado (muestran todo el log del sistema). | El respaldo falla, no se genera el archivo, o no se extraen los logs del sistema. | No hay evidencia de respaldo ni de trazabilidad mediante logs. |
| **8. Interacción Efectiva con IA y Trazabilidad** | 10% | Evidencia en Telegram de prompts estructurados (Contexto, Rol, Tarea). Exige y documenta las explicaciones de la IA. Entrega progresiva de evidencias por bloque. | Entrega en Telegram, pero los prompts carecen de estructura clara o no se evidencia la exigencia de explicaciones a la IA. | Entrega final en Telegram sin evidencia de proceso iterativo, o interacción con IA basada en preguntas sueltas y genéricas. | No hay evidencia de interacción con IA ni entregas progresivas en Telegram. |

### Instrucciones de Uso de la Rúbrica

1. **Validación de Trazabilidad (Criterio 8):** Antes de evaluar los artefactos finales, revise el historial del chat de Telegram. El estudiante debe haber subido las evidencias de cada Punto de Control conforme los completaba. Si la entrega es un "volcado" final de todos los archivos, el Criterio 8 debe calificarse como *Insuficiente*, independientemente de la calidad técnica del código.
2. **Auditoría del ETL (Criterio 4):** No basta con que las tablas tengan datos. Ejecute consultas de validación cruzada. Por ejemplo, cuente los registros únicos de `Tienda_Direccion` en el `reporte_crudo` y verifique que coincidan exactamente con el número de filas en la tabla `store` de la base de datos `maven_movies`.
3. **Evaluación de la Optimización (Criterio 5):** Solicite al estudiante que ejecute `EXPLAIN` en vivo sobre una de sus vistas. Si el estudiante no puede interpretar la salida (ej. no sabe qué significa `type: ALL` o `key: NULL`), el criterio debe bajar a *Insuficiente*, ya que la guía exige comprensión del plan de ejecución, no solo la ejecución del comando.
4. **Rigor en la Normalización (Criterio 3):** Verifique que el estudiante haya identificado las dependencias transitivas del reporte original (ej. `Tienda_ID` → `Tienda_Direccion`). Si el modelo lógico resultante mantiene columnas como `direccion` dentro de la tabla de transacciones (`rental` o `inventory`), el modelo no está en 3FN.
5. **Seguridad (Criterio 1):** Intente acceder a la máquina virtual usando la llave privada del estudiante. Si el acceso falla o el túnel SSH no redirige correctamente el puerto 3306 a `localhost` para DBeaver, el criterio no puede superar el nivel *Satisfactorio*.
