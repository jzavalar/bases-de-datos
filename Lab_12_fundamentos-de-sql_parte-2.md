### Laboratorio 12. Fundamentos del Lenguaje de Consulta Estructurada (SQL). Parte 2.

**Dr. Jesús Zavala Ruiz**   
**Creación:** 30 de junio de 2026.  
**Última actualización:** 13 de julio de 2026.  

---

#### Objetivo General

Consolidar las competencias avanzadas en el lenguaje de consulta estructurado (SQL) y la programación del lado del servidor en PostgreSQL, mediante la resolución de problemas complejos de extracción, manipulación y administración de datos sobre el esquema Pagila, integrando simultáneamente prácticas de auditoría forense y trazabilidad operativa, como preparación integral para el Examen Final de la unidad de enseñanza-aprendizaje.

#### Objetivos Específicos

1. **Dominio del orden de ejecución lógica:** Internalizar la distinción entre el orden sintáctico de escritura y el orden de procesamiento interno del motor relacional, aplicando correctamente las cláusulas `FROM`, `WHERE`, `GROUP BY`, `HAVING` y `SELECT` en consultas de complejidad creciente.

2. **Manipulación avanzada de conjuntos:** Desarrollar competencia en el uso de funciones de agregación, subconsultas correlacionadas, funciones de ventana (`ROW_NUMBER`, `LAG`, `SUM OVER`) y tipos de datos especializados (`tsrange`), preservando el detalle relacional cuando sea necesario.

3. **Combinación relacional y detección de anomalías:** Aplicar las distintas modalidades de `JOIN` para reconstruir información fragmentada, identificando tuplas huérfanas mediante el uso estratégico de `LEFT JOIN` y filtros `IS NULL`.

4. **Programación del lado del servidor:** Diseñar e implementar funciones, procedimientos almacenados y disparadores (triggers) en PL/pgSQL, aplicando control transaccional y encapsulando lógica de negocio directamente en el motor de base de datos.

5. **Administración y control de accesos:** Configurar roles, permisos granulares, vistas y mecanismos de respaldo, aplicando el principio de mínimo privilegio en la administración de sistemas de bases de datos.

6. **Trazabilidad y evidencia forense:** Configurar y operar sistemas de auditoría tanto a nivel del sistema operativo (`auditd` en Fedora 44) como del motor de base de datos (`postgresql.conf`), garantizando la no repudiación de operaciones mediante la extracción, verificación criptográfica y cadena de custodia de registros de auditoría.

7. **Preparación para la evaluación integral:** Desarrollar la autonomía necesaria para resolver problemas de bases de datos sin asistencia externa, fortaleciendo las competencias que serán evaluadas en el Examen Final teórico-práctico.

#### Competencias a Desarrollar

Al concluir el laboratorio, el estudiante será capaz de:

- **Analizar** esquemas relacionales complejos y formular consultas que integren múltiples tablas con operadores avanzados.
- **Diseñar** programas del lado del servidor que automaticen reglas de negocio y garanticen la integridad de los datos.
- **Administrar** sistemas de bases de datos PostgreSQL con criterios de seguridad y control de accesos.
- **Auditar** operaciones de base de datos mediante registros forenses verificables criptográficamente.
- **Depurar** consultas y procedimientos utilizando evidencia extraída de los logs del sistema.
- **Sustentar** técnicamente las decisiones de diseño e implementación ante un panel evaluador.

Se exhorta al estudiante a abordar este laboratorio con el mismo rigor que aplicará en el examen, utilizando la bitácora física, la evidencia forense y la defensa oral como mecanismos de verificación de su propio aprendizaje.

#### 1. Introducción

El presente laboratorio tiene como propósito evaluar y consolidar el dominio de los fundamentos del lenguaje SQL, abarcando desde la formulación de consultas básicas hasta la administración y programación del lado del servidor en **PostgreSQL**. Las prácticas se ejecutarán sobre la máquina virtual **Fedora44-lab**, configurada previamente en el Laboratorio 05. Para ello, se empleará **Pagila**, la adaptación nativa para PostgreSQL del clásico esquema Sakila (una simulación de una tienda de renta de películas). A diferencia de su contraparte original, Pagila introduce características avanzadas y propias del estándar PostgreSQL: tablas particionadas (como la tabla `payment`), rangos de tiempo (`tsrange` en la tabla `rental`), columnas generadas computacionalmente, y funciones de dominio público. Este esquema proporciona un entorno de complejidad real, diseñado para que el estudiante enfrente los mismos desafíos arquitectónicos que se resuelven en la industria.

Este laboratorio constituye un primer simulacro formal para el examen global. Dicho examen se llevará a cabo bajo condiciones de evaluación estrictas donde **estará prohibido el uso de herramientas de Inteligencia Artificial** (modelos de lenguaje, asistentes de código, etc.). El objetivo de esta restricción no es punitivo, sino pedagógico: la "fricción cognitiva" que experimenta el estudiante al diseñar consultas, depurar errores y comprender la lógica relacional sin asistencia externa es el mecanismo fundamental mediante el cual se consolidan las redes neuronales necesarias para el pensamiento analítico. Delegar este proceso a la IA impide el desarrollo de competencias reales.

Comprender el contexto económico y profesional de las bases de datos relacionales es fundamental. De acuerdo con el informe global de *The Business Research Company* (2026), el mercado de bases de datos relacionales alcanzó los **82.95 mil millones de dólares en 2025**, con proyecciones de crecimiento sostenido (CAGR del 10.8%) impulsadas por la migración a la nube y la analítica en tiempo real. 

En el contexto nacional, el Instituto Mexicano para la Competitividad (IMCO, 2024) y observatorios como Data México confirman que los perfiles capacitados en modelado de datos, optimización de consultas y administración de servidores se encuentran entre las profesiones con mayor remuneración y menor informalidad. Como advierte Boehm (1981), el costo de corregir un error de diseño de datos en producción es exponencialmente mayor que prevenirlo en la fase de modelado. El profesional que domina SQL no es un mero operador de software; es el arquitecto que garantiza la integridad de la información institucional. 

Dominar SQL representa una de las ventajas competitivas más sólidas para un estudiante de la Licenciatura en Computación que busca integrarse tempranamente al mercado laboral. De acuerdo con datos del Instituto Mexicano para la Competitividad (IMCO, 2024), las profesiones vinculadas a tecnologías de la información y administración de datos se encuentran entre las cinco mejor remuneradas del país, con salarios promedio mensuales que superan los 18,000 pesos para puestos de nivel junior y pueden alcanzar hasta los 45,000 pesos para especialistas con experiencia en optimización de consultas y modelado de datos. 

A nivel global, la Oficina de Estadísticas Laborales de Estados Unidos (BLS, 2024) proyecta que el empleo de administradores de bases de datos y arquitectos de datos crecerá un 12% entre 2022 y 2032, muy por encima del promedio de todas las ocupaciones. En México, plataformas como OCC Mundial reportan más de 3,000 vacantes activas para perfiles con conocimientos en SQL, PostgreSQL, MySQL y Oracle, abarcando sectores críticos como servicios financieros, telecomunicaciones, comercio electrónico y salud. 

Lo más relevante para un estudiante es que SQL es una habilidad "puerta de entrada": no requiere años de experiencia para ser útil. Desde el primer año de la carrera, un estudiante que domine consultas complejas, JOINs, funciones de ventana y administración básica de bases de datos puede acceder a posiciones de medio tiempo como analista de datos junior, desarrollador backend o administrador de bases de datos asistente, con remuneraciones que oscilan entre los 8,000 y 15,000 pesos mensuales. Esta temprana inserción no solo genera ingresos, sino que construye un portafolio de experiencia real que, al momento de graduarse, posiciona al profesional muy por encima de sus pares que carecen de práctica aplicada en entornos productivos.

#### 2. Metodología

Para garantizar la apropiación rigurosa de los conocimientos, el laboratorio se regirá por tres lineamientos ineludibles:

1. **Diseño Analítico Previo (La Bitácora Física):** La formulación de **todas** las consultas SQL debe realizarse previamente a mano en un cuaderno físico, incluyendo las correcciones progresivas. El estudiante debe diagramar la lógica, identificar las tablas, los predicados de unión (`JOIN`) y el orden de ejecución antes de transcribir el código a la terminal. El cuaderno escaneado es un entregable obligatorio que evidencia el proceso cognitivo, por lo que debe ser legible.  
2. **Restricción Absoluta de Inteligencia Artificial:** Queda estrictamente prohibido el uso de Modelos de Lenguaje (LLMs) para generar, depurar o explicar el código. La dependencia de estas herramientas anula el proceso de aprendizaje.  
3. **Trazabilidad y Evidencia Forense:** En la administración de sistemas, lo que no está registrado en los logs, no ocurrió. El estudiante deberá configurar y extraer los registros de auditoría tanto del sistema operativo (Fedora 44) como del motor de base de datos (PostgreSQL) para demostrar la autoría y ejecución de las operaciones.

#### 3. Preparación del Entorno en Fedora 44

#### 3.1. Verificación e Inicialización del Servicio

Acceda a su máquina virtual Fedora44-lab y asegure el funcionamiento del motor de base de datos:

```bash
ssh alumno@<ip_de_su_vm>
sudo systemctl status postgresql
```

Si el servicio muestra un error indicando que el directorio de datos no está inicializado:

```text
× postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: d>
    Active: failed (Result: exit-code) since Mon 2026-06-29 21:42:41 CST; 6min ago
    Process: 14683 ExecStartPre=/usr/libexec/postgresql-check-db-dir postgresql (co>
```

Ejecute el siguiente comando para inicializar la base de datos:

```bash
sudo /usr/bin/postgresql-setup --initdb
```

**Nota importante:** Si el directorio `/var/lib/pgsql/data` contiene archivos previos (incluso un directorio `log` vacío), el script de inicialización fallará. En ese caso, elimine el contenido residual:

```bash
sudo rm -rf /var/lib/pgsql/data/log
sudo /usr/bin/postgresql-setup --initdb
```

Debe obtener una salida similar a:

```text
 * Initializing database in '/var/lib/pgsql/data'
 * Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log
```

Posteriormente, inicie y habilite el servicio:

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
```

La salida esperada es:

```text
● postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: d>
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Mon 2026-06-29 21:54:19 CST; 2s ago
   Main PID: 15343 (postgres)
      Tasks: 7 (limit: 18959)
     Memory: 17.6M (peak: 18.4M)
        CPU: 145ms
     CGroup: /system.slice/postgresql.service
             ├─15343 /usr/bin/postgres -D /var/lib/pgsql/data
             ├─15345 "postgres: logger "
             ├─15346 "postgres: checkpointer "
             ├─15347 "postgres: background writer "
             ├─15349 "postgres: walwriter "
             ├─15350 "postgres: autovacuum launcher "
             └─15351 "postgres: logical replication launcher "
```

#### 3.2. Creación del Espacio de Trabajo

Proceda a crear el rol de usuario y la base de datos. Se solicita prestar especial atención a las credenciales:

```bash
sudo -u postgres psql
```

```sql
-- Obtener la lista de bases de datos
\list

-- Creación de la base de datos
CREATE DATABASE pagila;

-- Lista de usuarios:
\du

-- Creación del rol con contraseña
CREATE ROLE alumno WITH LOGIN PASSWORD 'uamIztapalapa';

\q
```

Conéctese posteriormente con el nuevo rol:

```bash
psql -U alumno -d pagila -h localhost
```

#### 3.3. Carga del Esquema y Datos de Pagila

Descargue los archivos oficiales desde el repositorio de GitHub y cárguelos respetando el orden lógico (primero la estructura (schema), luego los datos):

```bash
cd /tmp
wget https://github.com/devrimgunduz/pagila/archive/refs/heads/master.zip
unzip master.zip
cd pagila-master

sudo -u postgres psql -d pagila -f pagila-schema.sql
sudo -u postgres psql -d pagila -f pagila-data.sql
```

La salida debe mostrar múltiples operaciones `CREATE TABLE`, `CREATE INDEX`, `COPY` y `setval`:

```text
CREATE TABLE
ALTER TABLE
CREATE INDEX
...
COPY 599
COPY 6
COPY 1000
...
 setval 
--------
    599
(1 row)
```

Posteriormente, otorgue los permisos necesarios al usuario `alumno`:

```bash
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE pagila TO alumno;"
sudo -u postgres psql -d pagila -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO alumno;"
sudo -u postgres psql -d pagila -c "GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO alumno;"
```

Verifique que la base de datos fue creada correctamente:

```bash
sudo -u postgres psql -c "\l"
```

```text
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 pagila    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =Tc/postgres         +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres+
           |          |          |                 |             |             |            |           | alumno=CTc/postgres
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(4 rows)
```

#### 3.4. Configuración de la Auditoría del Sistema Operativo (El Perímetro)

Siguiendo las mejores prácticas de seguridad, primero se asegura el perímetro a nivel de kernel antes de configurar el motor de base de datos. El demonio de auditoría de Linux (`auditd`) registrará el acceso al sistema y las interacciones con los binarios de PostgreSQL.

```bash
# Instalación y activación del servicio de auditoría
sudo dnf install audit -y
```

```text
Updating and loading repositories:
 Fedora 44 - x86_64 - Updates               100% |  37.4 KiB/s |  14.2 KiB |  00m00s
Repositories loaded.
Package "audit-4.1.4-1.fc44.x86_64" is already installed.

Nothing to do.
```

```bash
# Habilitar auditd
sudo systemctl enable --now auditd

# Consultar estatus, start o restart, según se requiera 
sudo systemctl status auditd
sudo systemctl start auditd
sudo systemctl restart auditd
```

**Definición de reglas de auditoría para el laboratorio**

1. Monitorear conexiones de red iniciadas por el proceso de PostgreSQL  
```bash
sudo auditctl -a always,exit -F arch=b64 -S connect -F exe=/usr/bin/postgres -k postgresql_connections
```

2. Vigilar modificaciones en archivos de configuración y datos  
```bash
# Regla moderna para configuración (optimizada)
sudo auditctl -a always,exit -F arch=b64 -F path=/var/lib/pgsql/data/postgresql.conf -F perm=wa -k postgresql_config
```

```bash
# Regla moderna para directorio de datos (optimizada)
sudo auditctl -a always,exit -F arch=b64 -F path=/var/lib/pgsql/data -F perm=wa -k postgresql_data
```

3. Registrar la ejecución del cliente psql
```bash
sudo auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/psql -k psql_execution
```

#### 3.5. Configuración de la Auditoría Interna de PostgreSQL (El Motor)

Una vez asegurado el perímetro, se configura el motor para que registre cada sentencia SQL ejecutada, su duración y el contexto del usuario.

#### 3.5.1. Crear el directorio de logs

Antes de modificar la configuración, asegúrese de que el directorio de logs existe con los permisos correctos:

```bash
sudo mkdir -p /var/lib/pgsql/data/log
sudo chown postgres:postgres /var/lib/pgsql/data/log
sudo chmod 700 /var/lib/pgsql/data/log
```

#### 3.5.2. Editar el archivo de configuración

Edite el archivo de configuración:
```bash
sudo vi /var/lib/pgsql/data/postgresql.conf
```

Agregue las siguientes directivas **al final del archivo** (después de la sección `CUSTOMIZED OPTIONS`):

```ini
# Configuración de auditoría
log_statement = 'all'                  # Registra todas las sentencias SQL
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h ' # Contexto detallado
logging_collector = on                 # Activa el proceso colector de logs
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 7d                  # Rotación semanal de archivos
log_rotation_size = 100MB              # Rotación por tamaño
log_duration = on                      # Registra el tiempo de ejecución
log_min_duration_statement = 0         # Registra la duración de todas las sentencias
```

**Nota importante:** 
- Las unidades válidas para `log_rotation_age` son: `us`, `ms`, `s`, `min`, `h`, y `d`. La unidad `w` (semanas) **no es válida** en PostgreSQL.
- Las configuraciones personalizadas deben agregarse **al final del archivo** para evitar conflictos con directivas previas y facilitar la auditoría de cambios.

#### 3.5.3. Verificar permisos y contexto SELinux

Después de editar el archivo, verifique que los permisos y el contexto SELinux sean correctos. Este paso es **crítico** en Fedora 44, ya que SELinux puede bloquear el acceso a archivos incluso cuando los permisos UNIX son correctos.

```bash
# Verificar permisos
sudo ls -la /var/lib/pgsql/data/postgresql.conf
```

```text
-rw-------. 1 postgres postgres 33317 Jun 29 22:04 /var/lib/pgsql/data/postgresql.conf
```

```bash
# Si el propietario no es postgres, corregir:
sudo chown postgres:postgres /var/lib/pgsql/data/postgresql.conf
sudo chmod 600 /var/lib/pgsql/data/postgresql.conf

# Verificar contexto SELinux
sudo ls -laZ /var/lib/pgsql/data/postgresql.conf
```

```text
-rw-------. 1 postgres postgres unconfined_u:object_r:postgresql_db_t:s0 33317 Jun 29 22:04 /var/lib/pgsql/data/postgresql.conf
```

**Importante:** El contexto debe mostrar `postgresql_db_t:s0`. Si muestra `user_tmp_t:s0` u otro contexto incorrecto (común cuando se edita el archivo con `nano` o `vi`), restaure el contexto correcto:

```bash
sudo restorecon /var/lib/pgsql/data/postgresql.conf
```

#### 3.5.4. Configurar autenticación por contraseña

Por defecto, PostgreSQL en Fedora utiliza autenticación `peer`, que requiere que el usuario del sistema operativo coincida con el usuario de PostgreSQL. Para permitir la conexión del usuario `alumno`, modifique el archivo `pg_hba.conf`:

```bash
sudo vi /var/lib/pgsql/data/pg_hba.conf
```

Busque la línea:
```
local   all             all                                     peer
```

Y cámbiela a:
```
local   all             all                                     md5
```

Guarde el archivo y reinicie PostgreSQL:

```bash
sudo systemctl restart postgresql
```

#### 3.5.5. Reiniciar el servicio y verificar

Reinicie el servicio para aplicar todos los cambios:

```bash
sudo systemctl restart postgresql
sudo systemctl status postgresql
```

```text
● postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: d>
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Mon 2026-06-29 22:04:30 CST; 194ms ago
   Main PID: 16085 (postgres)
      Tasks: 7 (limit: 18959)
     Memory: 17.5M (peak: 18.8M)
        CPU: 188ms
     CGroup: /system.slice/postgresql.service
             ├─16085 /usr/bin/postgres -D /var/lib/pgsql/data
             ├─16086 "postgres: logger "
             ├─16087 "postgres: checkpointer "
             ├─16088 "postgres: background writer "
             ├─16090 "postgres: walwriter "
             ├─16091 "postgres: autovacuum launcher "
             └─16092 "postgres: logical replication launcher "
```

Verifique que el servicio esté activo (debe mostrar `active (running)` en verde).

#### 3.5.6. Verificar que los logs se están generando

```bash
sudo ls -lah /var/lib/pgsql/data/log/
```

```text
total 12K
drwx------. 1 postgres postgres  100 Jun 29 22:04 .
drwx------. 1 postgres postgres  588 Jun 29 22:04 ..
-rw-------. 1 postgres postgres  898 Jun 29 22:04 postgresql-2026-06-29_220430.log
-rw-------. 1 postgres postgres 2.2K Jun 29 22:04 postgresql-Mon.log
```

Debe mostrar archivos con el formato `postgresql-YYYY-MM-DD_HHMMSS.log`.

#### 3.5.7. Probar la auditoría

Ejecute una consulta de prueba:

```bash
psql -U alumno -d pagila -c "SELECT COUNT(*) FROM customer;"
```

Cuando se solicite la contraseña, ingrese: `uamIztapalapa`

```text
Password for user alumno: 
 count 
-------
   599
(1 row)
```

Verifique que la consulta fue registrada en el log:

```bash
sudo find /var/lib/pgsql/data/log/ -name "postgresql-*.log" -exec grep -H "SELECT COUNT" {} \;
```

Debe mostrar una entrada similar a:

```text
/var/lib/pgsql/data/log/postgresql-2026-06-29_220924.log:2026-06-29 22:10:01 CST [16355]: [1-1] user=alumno,db=pagila,app=psql,client=[local] LOG:  statement: SELECT COUNT(*) FROM customer;
```

Esto confirma que la auditoría está registrando correctamente:  
- Timestamp de la ejecución  
- PID del proceso PostgreSQL  
- Usuario que ejecutó la consulta  
- Base de datos consultada  
- Aplicación cliente  
- Dirección IP/origen del cliente  
- Sentencia SQL completa  

#### 4. Ejercicios del Laboratorio

Antes de comenzar los ejercicios, debe tener a la mano el diagrama entidad-relación de la base de datos:

```bash
cd ~
wget https://github.com/devrimgunduz/pagila/blob/master/pagila-schema-diagram.png
```

#### Parte 1: La Naturaleza Declarativa y el Orden de Ejecución Lógica
SQL es un lenguaje declarativo: el usuario especifica *qué* datos desea, y el motor determina *cómo* obtenerlos. Sin embargo, el orden sintáctico de escritura difiere del orden de ejecución lógica del motor. El motor procesa primero `FROM` (identifica las relaciones), luego `WHERE` (filtra tuplas), posteriormente `GROUP BY` (agrupa), y finalmente evalúa el `SELECT` (proyecta las columnas). Esta distinción ontológica explica por qué no es posible utilizar un alias definido en la cláusula `SELECT` dentro de la cláusula `WHERE`: al momento de evaluar el `WHERE`, el alias aún no existe en el contexto de ejecución.

1. **(4 pts)** Proyecte el nombre completo (`first_name` + `last_name`), correo electrónico y estado activo (`activebool`) de todos los clientes de la tabla `customer`. Ordene el resultado por apellido en orden ascendente.
2. **(4 pts)** Identifique las películas con clasificación (`rating`) 'G' o 'PG', cuya duración (`length`) exceda los 120 minutos y cuyo costo de renta (`rental_rate`) sea inferior a 3.00.
3. **(4 pts)** Liste las películas cuyo título contenga la cadena 'DINOSAUR' (insensible a mayúsculas) o finalice con el sufijo 'FIN'.
4. **(4 pts)** Seleccione los actores cuyo apellido inicie con una letra en el rango 'A' a 'D'. Ordene por apellido y nombre.
5. **(4 pts)** Obtenga el Top 10 de las películas con mayor duración (`length`).

#### Parte 2: Transición del Procesamiento Escalar a la Agregación Relacional
Las funciones de agregación (`SUM`, `COUNT`, `AVG`) transforman el procesamiento de tuplas individuales en el análisis de conjuntos. Aquí surge una confusión conceptual frecuente entre `WHERE` y `HAVING`. 
*   **`WHERE`** opera a nivel de tupla, *antes* de que existan los grupos. No puede evaluar funciones de agregación.
*   **`HAVING`** opera a nivel de grupo, *después* de que la agregación se ha computado. 

**Ejemplo didáctico:** Para encontrar clientes que han rentado más de 10 películas, la condición `COUNT(*) > 10` debe ir en el `HAVING`, no en el `WHERE`.

1. **(4 pts)** Calcule la duración promedio, mínima, máxima y la suma total de las duraciones de todas las películas.
2. **(4 pts)** Para cada clasificación (`rating`), muestre: la clasificación, el conteo de películas, la duración promedio y el costo de renta promedio.
3. **(4 pts)** Identifique las categorías que poseen más de 60 películas asignadas.
4. **(4 pts)** Cuente los clientes agrupados por la primera letra de su apellido. Ordene de mayor a menor frecuencia.
5. **(4 pts)** *Ejercicio de agregación condicional:* Utilizando la estructura `SUM(CASE WHEN ...)`, elabore un reporte por tienda (`store_id`) que muestre en columnas independientes: el total de clientes activos, el total de inactivos, y el total de clientes cuyo correo electrónico finaliza en 'sakilacustomer.org'.

#### Parte 3: Combinación de Relaciones y la Anatomía de los JOIN
El modelo relacional fragmenta la información para evitar redundancias. Los `JOIN` son los mecanismos que permiten reconstruir esta información. Internamente, un `LEFT JOIN` ejecuta tres pasos: (1) genera un producto cartesiano temporal entre ambas tablas, (2) filtra las filas donde el predicado `ON` es verdadero, y (3) **reincorpora** las tuplas de la tabla izquierda que no tuvieron coincidencia, rellenando las columnas de la tabla derecha con valores `NULL`. Este `NULL` es fundamental para detectar anomalías de integridad referencial (tuplas huérfanas).

1. **(5 pts)** Muestre el título de las películas y el nombre de su categoría (utilizando `INNER JOIN`).
2. **(5 pts)** Reconstruya la ruta geográfica completa del cliente: nombre, calle, distrito, ciudad y país (requiere encadenar 4 tablas).
3. **(5 pts)** Identifique los actores que han participado en más de 40 películas.
4. **(5 pts)** Utilizando `LEFT JOIN`, identifique las películas que **no** tienen registros de inventario físico (filtrando por `IS NULL`).

#### Parte 4: Descomposición Lógica mediante Subconsultas
Las subconsultas permiten anidar consultas para resolver problemas complejos paso a paso. Pueden devolver un valor escalar, una columna de valores (para usar con `IN`) o una tabla derivada. Tenga precaución con las subconsultas correlacionadas, ya que pueden impactar severamente el rendimiento si no existen índices adecuados.

1. **(5 pts)** Encuentre los clientes cuyo monto total de pagos exceda el promedio global de todos los pagos.
2. **(5 pts)** Liste las películas pertenecientes a las categorías 'Action' o 'Comedy' (empleando `IN` con subconsulta).
3. **(5 pts)** Identifique los actores que han participado en al menos una película con clasificación 'NC-17'.
4. **(5 pts)** Determine los clientes que han realizado más rentas que el promedio de rentas por cliente.

#### Parte 5: Funciones de Ventana y Preservación del Detalle Relacional
A diferencia del `GROUP BY`, que colapsa las filas, las **Funciones de Ventana** (`OVER`) permiten realizar cálculos de conjunto manteniendo intacto el detalle de cada tupla. Son indispensables para calcular rangos, acumulados o diferencias entre filas consecutivas. Adicionalmente, Pagila utiliza el tipo de dato `tsrange` para la tabla `rental`. Para extraer las fechas de inicio y fin de este rango, es imperativo utilizar las funciones nativas `lower()` y `upper()`.

1. **(5 pts)** Genere un ranking de clientes por tienda (`store_id`), ordenados por el monto total gastado, utilizando la función `ROW_NUMBER()`.
2. **(5 pts)** Calcule los días transcurridos entre una renta y la siguiente para cada cliente, utilizando la función `LAG()` sobre `lower(rental_period)`.
3. **(5 pts)** Muestre el monto acumulado de pagos por cliente, ordenado cronológicamente por fecha.

#### Parte 6: Manipulación de Datos y Responsabilidad Transaccional
Las operaciones del Lenguaje de Manipulación de Datos (DML: `INSERT`, `UPDATE`, `DELETE`) alteran el estado persistente de la base de datos. La omisión accidental de la cláusula `WHERE` en un `UPDATE` o `DELETE` puede resultar en la mutación masiva e irreversible de la información. Siempre verifique sus condiciones con un `SELECT` previo.

1. **(5 pts)** Inserte una nueva película titulada 'LABORATORIO SQL' (año 2026, idioma 1, 120 min, clasificación 'PG', renta 2.99).
2. **(5 pts)** Incremente el `replacement_cost` en un 15% exclusivamente para las películas de la categoría 'Horror'.
3. **(5 pts)** Elimine de la tabla `film_actor` cualquier registro donde el actor ya no exista en la tabla `actor`.

#### Parte 7: Programación del Lado del Servidor
PostgreSQL permite encapsular lógica de negocio directamente en el motor mediante PL/pgSQL. Esto mejora el rendimiento (al evitar tráfico de red), centraliza la seguridad y garantiza la consistencia de las reglas de negocio.

> **Nota de convención:** Siguiendo los estándares internacionales de desarrollo, los nombres de funciones, procedimientos y variables internas se escribirán **en inglés** (convención `snake_case`). Se incluye el significado en español en los comentarios.

**Tres ejemplos de Referencia:**
```sql
-- calculate_rental_days (calcular_días_de_renta)
CREATE OR REPLACE FUNCTION calculate_rental_days(p_rental_id INTEGER)
RETURNS INTEGER AS $$
DECLARE v_days INTEGER;
BEGIN
    SELECT EXTRACT(DAY FROM upper(rental_period) - lower(rental_period))::INTEGER
    INTO v_days FROM rental WHERE rental_id = p_rental_id;
    RETURN v_days;
END; $$ LANGUAGE plpgsql;

-- process_return (procesar_devolución) - Procedimiento con control transaccional
CREATE OR REPLACE PROCEDURE process_return(p_rental_id INTEGER)
LANGUAGE plpgsql AS $$
DECLARE v_rented_days INTEGER; v_fine NUMERIC := 0;
BEGIN
    v_rented_days := calculate_rental_days(p_rental_id);
    START TRANSACTION;
    UPDATE rental SET rental_period = tsrange(lower(rental_period), CURRENT_TIMESTAMP, '[)')
    WHERE rental_id = p_rental_id;
    IF v_rented_days > 5 THEN
        v_fine := (v_rented_days - 5) * 1.50;
        INSERT INTO payment (customer_id, staff_id, rental_id, amount, payment_date)
        SELECT customer_id, staff_id, rental_id, v_fine, CURRENT_TIMESTAMP FROM rental WHERE rental_id = p_rental_id;
    END IF;
    COMMIT;
END; $$;

-- audit_replacement_change (auditar_cambio_de_reemplazo) - Trigger de auditoría
CREATE OR REPLACE FUNCTION audit_replacement_change() RETURNS TRIGGER AS $$
BEGIN
    IF OLD.replacement_cost IS DISTINCT FROM NEW.replacement_cost THEN
        INSERT INTO film_audit (film_id, old_cost, new_cost)
        VALUES (NEW.film_id, OLD.replacement_cost, NEW.replacement_cost);
    END IF;
    RETURN NEW;
END; $$ LANGUAGE plpgsql;
```

**Ejercicios de Programación:**
1. **(7 pts)** **`get_customer_revenue` (obtener_ingresos_del_cliente):** Cree una función que reciba un `customer_id` y retorne el monto total gastado. Úsela en un `SELECT` para listar los 5 clientes con mayor gasto.
2. **(7 pts)** **`apply_category_discount` (aplicar_descuento_por_categoría):** Cree un procedimiento que reciba un nombre de categoría y un porcentaje, y aplique dicho descuento al `rental_rate` de las películas correspondientes.
3. **(6 pts)** **Trigger de Auditoría de Pagos:** Cree una tabla `payment_audit` y un trigger que registre cada inserción en la tabla `payment`, almacenando `payment_id`, `customer_id`, `amount`, `payment_date` y `changed_by` (utilizando `CURRENT_USER`).

#### Parte 8: Administración de Base de Datos y Control de Accesos
1. **(5 pts)** Cree un rol llamado `analista_ventas` con contraseña 'uamIztapalapa'.
2. **(5 pts)** Otorgue al rol `analista_ventas` únicamente el permiso `SELECT` sobre las tablas `customer`, `payment` y `rental`.
3. **(5 pts)** Cree una vista llamada `vista_clientes_activos` que integre los datos geográficos completos de los clientes activos.
4. **(5 pts)** Genere un respaldo exclusivo del esquema (estructura) de la base de datos Pagila utilizando `pg_dump -s` y guárdelo como `esquema_pagila.sql`.

#### 5. Extracción y Preservación de Evidencia Forense

Al concluir las prácticas, el estudiante deberá extraer los registros de auditoría para conformar su cadena de custodia. Para ello, se definirá un rango de fechas que abarque desde el inicio hasta la finalización del laboratorio, garantizando que toda la evidencia generada durante las sesiones sea capturada de forma íntegra y verificable. El proceso se realiza en dos fases: primero en la **máquina virtual** (donde se generan las evidencias), y posteriormente en el **equipo anfitrión** (donde se verifican y empaquetan para su entrega).

#### 5.1. Definición del período de análisis (en la VM)

Dentro de la máquina virtual, el estudiante debe configurar las variables de fecha. **Importante:** Los comandos `ausearch` y `journalctl` utilizan formatos de fecha diferentes:

- **`ausearch`** requiere formato `MM/DD/YYYY`
- **`journalctl`** acepta formato `YYYY-MM-DD`

```bash
# Dentro de la VM Fedora44-lab, como usuario alumno

# Para ausearch (formato MM/DD/YYYY)
FECHA_INICIO_AUSEARCH="06/25/2026"
FECHA_FIN_AUSEARCH="06/30/2026"

# Para journalctl (formato YYYY-MM-DD)
FECHA_INICIO="2026-06-25"
FECHA_FIN="2026-06-30"

echo "Período de análisis: $FECHA_INICIO a $FECHA_FIN"
```

> **Nota:** La fecha de inicio debe corresponder al día en que se configuró la auditoría, y la fecha de fin al día en que se realiza la extracción de evidencia. Ajuste los valores según las fechas reales del laboratorio.

#### 5.2. Verificación de prerequisitos (en la VM)

Antes de ejecutar la extracción, verifique que los servicios de auditoría están activos y que las reglas de `auditd` están configuradas:

```bash
# Verificar que PostgreSQL está activo
sudo systemctl status postgresql | grep "Active:"

# Verificar que auditd está activo
sudo systemctl status auditd | grep "Active:"

# Verificar reglas de auditd (debe mostrar al menos 4 reglas)
sudo auditctl -l
```

La salida esperada de `auditctl -l` debe incluir las reglas:
- `postgresql_connections`
- `postgresql_config`
- `postgresql_data`
- `psql_execution`

Si alguna regla falta, reconfigúrela siguiendo las instrucciones de la sección 3.4.

#### 5.3. Extracción de evidencias (en la VM)

Ejecute el siguiente bloque de comandos **dentro de la máquina virtual** para extraer los registros de las tres fuentes de auditoría:

```bash
# Dentro de la VM Fedora44-lab

# 1. Evidencia del Sistema Operativo (Perímetro)
sudo journalctl -u sshd --since "$FECHA_INICIO" --until "$FECHA_FIN" > ~/logs_ssh.txt
sudo ausearch -k postgresql_connections --start "$FECHA_INICIO_AUSEARCH" --end "$FECHA_FIN_AUSEARCH" > ~/logs_auditd_postgres.txt
sudo ausearch -k psql_execution --start "$FECHA_INICIO_AUSEARCH" --end "$FECHA_FIN_AUSEARCH" > ~/logs_auditd_psql.txt

# 2. Evidencia del Motor de Base de Datos (Núcleo)
# Usar sudo bash -c para que el glob se expanda con privilegios de root
sudo bash -c 'cat /var/lib/pgsql/data/log/postgresql-*.log > /home/alumno/logs_postgresql.txt'
sudo chown alumno:alumno ~/logs_postgresql.txt

# 3. Generar una consulta de prueba para asegurar evidencia en auditd
psql -U alumno -d pagila -c "SELECT 'AUDIT_EVIDENCE_TEST' AS test_message;"

# Re-extraer logs de auditd (ahora incluirá la consulta de prueba)
sudo ausearch -k psql_execution --start "$FECHA_INICIO_AUSEARCH" --end "$FECHA_FIN_AUSEARCH" > ~/logs_auditd_psql.txt

# 4. Sellado de Integridad (Hashes SHA-256)
sha256sum ~/logs_*.txt > ~/hashes_evidencias.txt

# 5. Registro de la cadena de custodia (VM)
cat > ~/cadena_custodia.txt << EOF
=== REGISTRO DE CADENA DE CUSTODIA (VM) ===
Fecha de extracción: $(date '+%Y-%m-%d %H:%M:%S %Z')
Período analizado:   $FECHA_INICIO a $FECHA_FIN
Responsable:         $(whoami)
Equipo:              $(hostname)

$(cat ~/hashes_evidencias.txt)
EOF
```

#### 5.4. Verificación de la extracción (en la VM)

Una vez ejecutados los comandos, verifique que todas las evidencias fueron generadas correctamente **dentro de la VM**:

```bash
# Listar los archivos generados con su tamaño
ls -lh ~/logs_*.txt ~/hashes_evidencias.txt ~/cadena_custodia.txt

# Visualizar los hashes generados
cat ~/hashes_evidencias.txt

# Contar el número de registros por fuente
wc -l ~/logs_*.txt
```

La salida esperada debe mostrar archivos con contenido (tamaño mayor a 0 bytes):

```text
-rw-r--r--. 1 alumno alumno  592 Jun 29 23:47 cadena_custodia.txt
-rw-r--r--. 1 alumno alumno  395 Jun 29 23:47 hashes_evidencias.txt
-rw-r--r--. 1 alumno alumno  215 Jun 29 23:47 logs_auditd_postgres.txt
-rw-r--r--. 1 alumno alumno  414 Jun 29 23:47 logs_auditd_psql.txt
-rw-r--r--. 1 alumno alumno 181K Jun 29 23:47 logs_postgresql.txt
-rw-r--r--. 1 alumno alumno 4.2K Jun 29 23:47 logs_ssh.txt
```

> **Importante:** Si `logs_auditd_psql.txt` está vacío (0 bytes), significa que las reglas de auditd se configuraron después de ejecutar las consultas. Ejecute la consulta de prueba del paso 5.3 y re-extraiga los logs.

#### 5.5. Identificación de la IP de la VM (en el host)

Antes de transferir las evidencias al equipo anfitrión, identifique la dirección IP de la máquina virtual. Desde el **host Fedora 44**, ejecute:

```bash
# Desde el host Fedora 44

# Método 1: Usar domifaddr (recomendado)
sudo virsh -c qemu:///system domifaddr fedora44-lab

# Método 2: Usar los leases DHCP de la red default
sudo virsh -c qemu:///system net-dhcp-leases default
```

La salida esperada es similar a:

```text
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet0      52:54:00:3a:eb:41    ipv4         192.168.122.24/24
```

Anote la dirección IP (en este ejemplo, `192.168.122.24`).

#### 5.6. Transferencia de evidencias al host (desde el host)

Una vez identificada la IP de la VM, transfiera las evidencias al equipo anfitrión usando SCP. Desde el **host Fedora 44**, ejecute:

```bash
# Desde el host Fedora 44

# Configurar la IP de la VM (reemplazar con la IP real obtenida en el paso 5.5)
VM_IP="192.168.122.24"

# Verificar conectividad
ping -c 3 $VM_IP

# Crear directorio de destino
mkdir -p ~/Laboratorio12_Evidencias

# Transferir archivos por SCP (pedirá la contraseña del usuario alumno)
scp alumno@$VM_IP:~/logs_*.txt ~/Laboratorio12_Evidencias/
scp alumno@$VM_IP:~/hashes_evidencias.txt ~/Laboratorio12_Evidencias/
scp alumno@$VM_IP:~/cadena_custodia.txt ~/Laboratorio12_Evidencias/
```

La salida esperada muestra el progreso de cada transferencia:

```text
logs_auditd_postgres.txt                          100%  215    80.9KB/s   00:00    
logs_auditd_psql.txt                              100%  414   157.9KB/s   00:00    
logs_postgresql.txt                               100%  181KB  14.8MB/s   00:00    
logs_ssh.txt                                      100% 4273     1.6MB/s   00:00    
hashes_evidencias.txt                             100%  395   122.4KB/s   00:00    
cadena_custodia.txt                               100%  592   175.3KB/s   00:00    
```

#### 5.7. Verificación de integridad en el host

**Punto crítico:** El archivo `hashes_evidencias.txt` transferido desde la VM contiene rutas absolutas (`/home/alumno/...`) que no existen en el host. Por lo tanto, es necesario **regenerar los hashes en el host** usando rutas relativas para validar la integridad de la transferencia.

```bash
# Desde el host Fedora 44

# Posicionarse en el directorio de evidencias
cd ~/Laboratorio12_Evidencias/

# Listar los archivos recibidos
ls -lh

# Regenerar los hashes usando rutas relativas (solo el nombre del archivo)
sha256sum logs_*.txt > hashes_evidencias_host.txt

# Ver el contenido de los nuevos hashes
cat hashes_evidencias_host.txt

# Verificar la integridad
sha256sum -c hashes_evidencias_host.txt
```

La salida esperada debe mostrar `OK` para cada archivo:

```text
b1e3a37f73e46c1d2554d45aedcd2d4a44df12522076dd824436c5eac8d51ca0  logs_auditd_postgres.txt
769a48f35dfe753a0ace82d92a347f09442cd6f6eea87b0f5e39ff0bb850fe6e  logs_auditd_psql.txt
1f18e1d204a04d1e56b4cad4613ca8cabbf4f04f1a974d20e8ecc68c3b03c1e7  logs_postgresql.txt
3289673c8010c40b2e746e06decdfee2d18b5f13a099cacaa73e1de9dac3f777  logs_ssh.txt

logs_auditd_postgres.txt: OK
logs_auditd_psql.txt: OK
logs_postgresql.txt: OK
logs_ssh.txt: OK
```

> **Importante:** Si algún archivo muestra `FAILED`, la transferencia se corrompió y debe repetirse desde el paso 5.6.

#### 5.8. Registro de la cadena de custodia en el host

Para garantizar la trazabilidad forense completa, el estudiante debe documentar la recepción y verificación de las evidencias en el equipo anfitrión:

```bash
# Desde el host Fedora 44, dentro del directorio de evidencias
cd ~/Laboratorio12_Evidencias/

cat > cadena_custodia_host.txt << EOF
=== REGISTRO DE CADENA DE CUSTODIA (HOST) ===
Fecha de verificación: $(date '+%Y-%m-%d %H:%M:%S %Z')
Responsable:           $(whoami)
Equipo anfitrión:      $(hostname)
Directorio:            $(pwd)
IP de la VM origen:    $VM_IP

$(cat hashes_evidencias_host.txt)
EOF

cat cadena_custodia_host.txt
```

La salida esperada es similar a:

```text
=== REGISTRO DE CADENA DE CUSTODIA (HOST) ===
Fecha de verificación: 2026-06-29 23:52:26 CST
Responsable:           alumno
Equipo anfitrión:      fedora44-host
Directorio:            /home/alumno/Laboratorio12_Evidencias
IP de la VM origen:    192.168.122.24

b1e3a37f73e46c1d2554d45aedcd2d4a44df12522076dd824436c5eac8d51ca0  logs_auditd_postgres.txt
769a48f35dfe753a0ace82d92a347f09442cd6f6eea87b0f5e39ff0bb850fe6e  logs_auditd_psql.txt
1f18e1d204a04d1e56b4cad4613ca8cabbf4f04f1a974d20e8ecc68c3b03c1e7  logs_postgresql.txt
3289673c8010c40b2e746e06decdfee2d18b5f13a099cacaa73e1de9dac3f777  logs_ssh.txt
```

#### 5.9. Empaquetado final de la evidencia

Una vez verificada la integridad y documentada la cadena de custodia, comprima todas las evidencias en un archivo único para su entrega:

```bash
# Desde el host Fedora 44, dentro del directorio de evidencias
cd ~/Laboratorio12_Evidencias/

# Crear archivo comprimido con todas las evidencias
tar -czf ~/Laboratorio12_Evidencias_$(date +%Y%m%d).tar.gz \
    logs_*.txt hashes_evidencias_host.txt cadena_custodia_host.txt

# Verificar el contenido del archivo comprimido
tar -tzf ~/Laboratorio12_Evidencias_$(date +%Y%m%d).tar.gz

# Verificar el tamaño del archivo generado
ls -lh ~/Laboratorio12_Evidencias_*.tar.gz
```

La salida esperada del `tar -tzf` debe mostrar:

```text
logs_auditd_postgres.txt
logs_auditd_psql.txt
logs_postgresql.txt
logs_ssh.txt
hashes_evidencias_host.txt
cadena_custodia_host.txt
```

#### 5.10. Cierre de la sesión SSH con la máquina virtual

Una vez transferidas y verificadas las evidencias en el host, cierre correctamente la sesión SSH con la máquina virtual para liberar recursos y mantener la seguridad del entorno:

```bash
# Desde el host Fedora 44

# Verificar si hay sesiones SSH activas hacia la VM
w | grep $VM_IP

# Si hay sesiones activas, cerrarlas
pkill -f "ssh.*$VM_IP"

# Verificar que no queden sesiones
w | grep $VM_IP || echo "No hay sesiones activas hacia la VM"
```

> **Importancia forense:** El cierre limpio de la sesión SSH deja constancia en los logs de auditoría (`logs_ssh.txt`) del momento exacto en que concluyó la intervención, reforzando la cadena de custodia. Una sesión SSH abandonada puede generar entradas espurias en los logs que comprometan la trazabilidad del laboratorio.

#### 5.11. Notas importantes

1. **Flujo de trabajo en dos fases:** La extracción de evidencias se realiza en **dos fases**: primero en la VM (donde se generan los archivos) y después en el host (donde se verifican y empaquetan). No intente ejecutar los comandos de extracción directamente en el host.

2. **Formato de fechas:** Los comandos `ausearch` y `journalctl` utilizan formatos de fecha diferentes. El script maneja esta diferencia mediante variables separadas (`FECHA_INICIO_AUSEARCH` en formato `MM/DD/YYYY` y `FECHA_INICIO` en formato `YYYY-MM-DD`).

3. **Reglas de auditd:** Las reglas de auditoría deben estar configuradas **antes** de ejecutar las consultas que desea capturar. Verifique que todas las reglas estén activas con `sudo auditctl -l`.

4. **Archivos vacíos:** Si `logs_auditd_psql.txt` está vacío (0 bytes), significa que las reglas se configuraron después de ejecutar las consultas. Ejecute una consulta de prueba adicional y re-extraiga los logs.

5. **Extracción de logs de PostgreSQL:** El directorio `/var/lib/pgsql/data/log/` requiere privilegios de superusuario. Use `sudo bash -c` para que el glob `*.log` se expanda con los permisos adecuados.

6. **Regeneración de hashes en el host:** El archivo `hashes_evidencias.txt` original contiene rutas absolutas de la VM (`/home/alumno/...`) que no existen en el host. Es **obligatorio** regenerar los hashes en el host usando rutas relativas (`logs_*.txt`) para validar la integridad de la transferencia.

7. **Integridad criptográfica:** Los hashes SHA-256 deben calcularse inmediatamente después de la extracción y conservarse en un medio separado. Cualquier modificación posterior a los archivos de log invalidará los hashes, lo cual constituye un indicador de alteración de la evidencia.

8. **Confidencialidad:** Los archivos de log contienen información sensible de su infraestructura (usuarios, IPs, consultas SQL). No suba el archivo comprimido a repositorios públicos ni lo comparta por medios no cifrados.

9. **Cierre limpio:** Siempre cierre las sesiones SSH antes de apagar o suspender la VM. Las sesiones SSH abiertas pueden dejar procesos residuales que consuman recursos y generen entradas espurias en los logs de auditoría.

10. **Respaldo adicional:** Se recomienda crear una copia de seguridad del archivo comprimido en un medio extraíble o ubicación segura para su posterior análisis forense.


#### 6. Entregables y Formato de Envío

Todo el material deberá integrarse y comprimirse en un archivo denominado `Laboratorio12_ApellidoNombre.zip`, conteniendo:

1. **La Bitácora Analógica:** Fotografías o escaneos legibles del cuaderno físico con el diseño de *todas* las consultas.
2. **El Script SQL:** Archivo `laboratorio12_pagila.sql` con el código final, estructurado y comentado.
3. **Las Bitácoras de Auditoría:** Archivos `logs_ssh.txt`, `logs_auditd_postgres.txt`, `logs_auditd_psql.txt` y `logs_postgresql.txt`.
4. **El Certificado de Integridad:** Archivo `hashes_evidencias.txt`.
5. **El Informe Académico (PDF):** Documento `laboratorio12_informes.pdf` que incluya las respuestas al cuestionario teórico, la rúbrica de autoevaluación y el ensayo reflexivo.

#### 7. Cuestionario de Fundamentos Teóricos

En su informe PDF, responda a las siguientes preguntas con rigor técnico y claridad conceptual:

1. Explique la diferencia entre el orden sintáctico de escritura y el orden de ejecución lógica de una consulta `SELECT`. ¿Por qué el motor rechaza la consulta `SELECT COUNT(*) AS total FROM rental WHERE total > 5 GROUP BY customer_id`?
2. Describa con sus propias palabras los tres pasos internos que ejecuta el motor al procesar un `LEFT JOIN`. ¿Por qué la aparición de valores `NULL` en las columnas de la tabla derecha es información valiosa para la auditoría de datos?
3. La tabla `payment` en Pagila está particionada por rangos de fechas. Si se ejecuta `SELECT * FROM payment WHERE payment_date = '2007-02-15'`, ¿qué mecanismo utiliza el motor para evitar leer las particiones de enero o marzo?
4. ¿Qué implicaciones técnicas tiene intentar ejecutar `UPDATE film SET rentals_to_breakeven = 50`? (Analice la definición de la columna en el esquema).
5. Establezca la diferencia práctica y forense entre los registros generados por `postgresql.conf` y los eventos capturados por `auditd` / `journalctl` en Fedora 44.
6. ¿Cuál es el propósito criptográfico y legal de generar hashes SHA-256 de los archivos de log antes de su entrega?

#### 8. Rúbrica de Autoevaluación de Competencias

*Marque las siguientes casillas en su PDF únicamente si es capaz de explicar el concepto en voz alta sin consultar material de apoyo.*

**Competencias Técnicas:**
- [ ] Comprendo y puedo dibujar el orden de ejecución lógica de un `SELECT`.
- [ ] Puedo justificar la diferencia ontológica entre `WHERE` (filtra tuplas) y `HAVING` (filtra grupos).
- [ ] Entiendo la anatomía de un `LEFT JOIN` y la utilidad del `NULL` para detectar huérfanos.
- [ ] Sé manipular rangos de tiempo (`tsrange`) extrayendo sus límites con `lower()` y `upper()`.
- [ ] Comprendo el propósito y la sintaxis de las Funciones de Ventana (`OVER`, `PARTITION BY`).
- [ ] Puedo diferenciar cuándo utilizar una Función, un Procedimiento Almacenado o un Disparador (Trigger).

**Competencias Administrativas:**
- [ ] Sé configurar `auditd` en Linux y `postgresql.conf` para garantizar la trazabilidad completa.
- [ ] Entiendo la diferencia estructural entre respaldar el esquema (`pg_dump -s`) y los datos (`pg_dump -a`).

**Integridad Académica:**
- [ ] Diseñé la totalidad de las consultas en papel antes de interactuar con el terminal.
- [ ] Comprendo que cada interacción con el servidor deja una huella forense innegable e inmutable.

#### 9. Ensayo Reflexivo sobre el Proceso de Aprendizaje

En su informe PDF, redacte un ensayo de al menos 500 palabras abordando los siguientes ejes de metacognición:

1. **La Fricción Cognitiva:** Identifique en qué ejercicios experimentó mayor dificultad conceptual (ej. manipulación de `tsrange`, encadenamiento de 4 tablas, diferencia entre `WHERE` y `HAVING`). ¿Cómo resolvió estos obstáculos mediante el razonamiento analítico?
2. **El Diseño Analógico vs. Digital:** ¿De qué manera la obligación de dibujar las relaciones y la lógica en un cuaderno físico modificó su proceso de pensamiento en comparación con depender del autocompletado de un editor?
3. **Depuración y Evidencia:** Relate algún error de sintaxis o lógica que haya cometido y explique cómo utilizó los logs de PostgreSQL o de `auditd` para diagnosticar y solucionar el problema.
4. **El Impacto de la IA en el Desarrollo Cognitivo:** Reflexione críticamente sobre por qué delegar el pensamiento lógico a un Modelo de Lenguaje en esta etapa de su formación lo convertiría en un profesional vulnerable y fácilmente reemplazable en el mercado laboral actual.
5. **Conciencia Forense:** ¿Qué le hizo sentir al ver su propio usuario, su IP y sus consultas registrados en los logs del sistema operativo? ¿Cómo modifica esta experiencia su perspectiva sobre la privacidad y la seguridad en la administración de servidores institucionales?

#### 10. Recomendaciones Metodológicas

1. **Exploración del Catálogo de Datos:** Antes de formular consultas, utilice los metacomandos `\dt` y `\d+ nombre_tabla` en `psql`. El esquema de Pagila contiene restricciones, índices y vistas que revelan la topología de las relaciones.
2. **Análisis de Vistas Nativas:** Pagila incluye vistas complejas como `customer_list` o `sales_by_film_category`. Estudie su código fuente para comprender cómo los ingenieros originales resolvieron problemas de agregación y unión.
3. **Interpretación de los Logs:** Los registros de PostgreSQL incluyen el tiempo de ejecución (`log_duration`). Aprenda a identificar qué consultas son ineficientes.
4. **Confidencialidad de la Evidencia:** Los archivos de log contienen información sensible de su infraestructura. No suba el archivo ZIP a repositorios públicos.

#### 11. Cronograma y Políticas de Evaluación

**Fecha límite de entrega:** Viernes 17 de julio de 2026, 16:00 horas.

*   **Trabajo estrictamente individual.**
*   **Integridad Académica:** La detección de patrones generados por IA, o la ausencia de la bitácora física (cuaderno), resultará en la anulación automática del laboratorio y el inicio del procedimiento correspondiente por falta a la ética académica.
*   **Entregas tardías:** Se aplicará una penalización del 10% por cada día de retraso.
*   **Evidencia incompleta:** La ausencia de los archivos de log o de los hashes de integridad se calificará como trabajo no entregado.

Se exhorta a los estudiantes a abordar este laboratorio con el rigor que exige la ingeniería de datos. La fricción que experimentan hoy en el diseño lógico es la competencia que garantizará su excelencia profesional mañana.

#### 12. Sesión de Defensa Oral y Verificación de Competencias

**Fecha:** Lunes 20 y Miércoles 22 de julio de 2026, en el horario regular de la clase.

Con el propósito de garantizar la integridad académica y verificar la apropiación real de las competencias desarrolladas durante el laboratorio, se llevará a cabo una **sesión de defensa oral individual** durante las sesiones de los días 20 y 22 de julio de 2026. Cada estudiante deberá presentarse con su cuaderno físico (bitácora analógica) y su script SQL (`laboratorio12_pagila.sql`) para participar en un ejercicio de verificación de 30 minutos, durante el cual el profesor formulará preguntas sobre:

1. **Explicación del razonamiento lógico:** El estudiante deberá explicar, en sus propias palabras, la lógica detrás de al menos tres consultas de su elección, identificando el orden de ejecución y justificando el uso de `JOIN`, `GROUP BY`, `HAVING` o funciones de ventana.

2. **Modificación en vivo:** A partir de una consulta de su propio script, el profesor solicitará una modificación específica (por ejemplo, agregar un filtro, cambiar un `INNER JOIN` por un `LEFT JOIN`, o incorporar una función de agregación). El estudiante deberá realizar el cambio en la terminal y explicar el resultado.

3. **Relación con la bitácora física:** El estudiante deberá identificar en su cuaderno la consulta donde realizó más correcciones o tachaduras, y explicar qué error conceptual cometió inicialmente y cómo lo resolvió.

4. **Interpretación de evidencia forense:** A partir de los archivos de log entregados, el estudiante deberá identificar una consulta específica ejecutada por él, señalar el timestamp, el usuario y el PID registrado, y explicar cómo esta evidencia garantiza la trazabilidad de su trabajo.

5. **Fundamentos teóricos:** Se formularán entre dos y tres preguntas del cuestionario teórico (sección 7), seleccionadas aleatoriamente, que el estudiante deberá responder oralmente sin consulta de material.

**Importante:** La sesión de defensa oral constituye parte importente de la calificación final del laboratorio. La incapacidad de explicar el código propio, la ausencia de la bitácora física, o la evidencia de que el estudiante no comprendió las consultas presentadas (independientemente de si fueron generadas con asistencia de IA o no), resultará en la anulación de esta porción de la calificación. El objetivo de esta evaluación no es punitivo, sino pedagógico: garantizar que las competencias declaradas en la rúbrica de autoevaluación (sección 8) hayan sido efectivamente desarrolladas, y que el estudiante esté en capacidad de demostrarlo en un entorno profesional real donde la comprensión profunda del código es indispensable.

Orden de Presentacion de la Evaluación Global, según el número de matrícula, calculado con R: 

```r
# Establecer la semilla fija
set.seed(123)

# Crear el vector con los números proporcionados
numeros <- c(1:10)

matriculas <- c(2213042983, 2243051696, 2222018735, 2193055660, 2243050957, 
             2213042572, 2203009190, 2243009245, 2193055777, 2243012733)

# Ordenar aleatoriamente los números
evaluacion <- sample(matriculas)

# Mostrar el resultado
print(evaluacion)
 [1] 2222018735 2243012733 2243051696 2243009245 2213042572
 [6] 2193055777 2213042983 2203009190 2243050957 2193055660
```
Horario de presentación de la Evaluación Global:

| Matrícula     | Día               | Fecha               | Hora de inicio | Hora de fin |
| --------------|-------------------|---------------------|----------------|-------------|
| 2222018735    | Lunes             | 20 de julio        | 16:00          | 16:30       |
| 2243012733    | Lunes             | 20 de julio        | 16:30          | 17:00       |
| 2243051696    | Lunes             | 20 de julio        | 17:00          | 17:30       |
| 2243009245    | Lunes             | 20 de julio        | 17:30          | 18:00       |
| 2213042572    | Lunes             | 20 de julio        | 18:00          | 18:30       |
| 2193055777    | Miércoles         | 22 de julio        | 16:00          | 16:30       |
| 2213042983    | Miércoles         | 22 de julio        | 16:30          | 17:00       |
| 2243050957    | Miércoles         | 22 de julio        | 17:00          | 17:30       |
| 2193055660    | Miércoles         | 22 de julio        | 17:30          | 18:00       |

Los estudiantes que no puedan asistir a la sesión del 20 o 22 de julio deberán presentar la Evaluación de Recuperación.

#### 13. Referencias Bibliográficas

Boehm, B. W. (1981). *Software engineering economics*. Prentice-Hall.

Data México. (s.f.). *Observatorio de complejidad económica y laboral*. Secretaría de Economía de México. https://datamexico.org/es/

Evans, J. (2024). *SQL* [Zine]. https://jvns.ca/

Instituto Mexicano para la Competitividad. (2024). *Compara Carreras: Herramienta de análisis de educación y mercado laboral*. https://comparacarreras.imco.org.mx/

OCC Mundial. (2026). *Bolsa de trabajo: Tecnología e Informática*. https://www.occ.com.mx/empleos/de-informatica-y-tecnologia/

Stack Overflow. (2025). *2025 Developer Survey: Technology section*. https://survey.stackoverflow.co/

The Business Research Company. (2026). *Relational Database Global Market Report 2026*. https://www.thebusinessresearchcompany.com/report/relational-database-global-market-report

U.S. Bureau of Labor Statistics. (2024a). *Database administrators and architects: Occupational Outlook Handbook*. https://www.bls.gov/ooh/computer-and-information-technology/database-administrators.htm

U.S. Bureau of Labor Statistics. (2024b). *Computer systems analysts: Occupational Outlook Handbook*. https://www.bls.gov/ooh/computer-and-information-technology/computer-systems-analysts.htm
