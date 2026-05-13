[p. 1]

### CAPÍTULO 1. Arquitectura del Sistema PostgreSQL

**Fuente:** Kumar, Y. V. R., Samayam, A. K., & Kadambari, P. (2025). *Mastering PostgreSQL Administration: Internals, Operations, Monitoring, and Oracle Migration Strategies*. Apress/Springer.

#### Introducción

En este capítulo, aprenderemos sobre el origen de PostgreSQL, su arquitectura y sus componentes, y comprenderemos los diversos componentes de PostgreSQL durante una solicitud de conexión de cliente.

Cubrimos los siguientes temas:

- Origen de PostgreSQL

- Componentes arquitectónicos clave de PostgreSQL

- Árbol de procesos del servidor PostgreSQL

- Flujo de arquitectura de PostgreSQL

PostgreSQL es uno de los sistemas de gestión de bases de datos objeto-relacionales (ORDBMS, por sus siglas en inglés) de código abierto más populares. Se basa en POSTGRES, Versión 4.2, desarrollado inicialmente en el departamento de Ciencias de la Computación de la Universidad de California en Berkeley. PostgreSQL se basa en el código de Berkeley y sigue el estándar SQL, ofreciendo diversas características que cubriremos en este libro.

La arquitectura de PostgreSQL está diseñada para manejar transacciones de manera eficiente y mantener la integridad de los datos, soportando sesiones concurrentes. En este capítulo, aprenderemos sobre los diversos componentes de la arquitectura interna de PostgreSQL y sus funciones.

PostgreSQL sigue una arquitectura típica cliente/servidor. El cliente y el servidor usualmente se encuentran en hosts diferentes y se comunican a través de una conexión de red TCP/IP. Varios componentes centrales trabajan conjuntamente para manejar consultas, gestionar el almacenamiento de datos y asegurar la ejecución eficiente de las operaciones de consulta.

[p. 2]

#### Componentes de PostgreSQL

Los componentes clave en una arquitectura de PostgreSQL son:

1.  Proceso del servidor

2.  Proceso de utilidad

3.  Memoria del sistema

4.  Archivos físicos

Cada uno de estos componentes centrales depende además de varios componentes menores. Aprendamos sobre cada componente central en detalle y su función.

##### **1. Proceso del servidor**

Un proceso del servidor consiste en los siguientes componentes:

**Proceso postmaster:** Este es el proceso central que inicia y controla todos los demás procesos de PostgreSQL. Escucha las conexiones entrantes de clientes y genera procesos en segundo plano para manejar cada conexión.

**Proceso en segundo plano postgres:** Cada conexión entrante de cliente es atendida por un proceso backend dedicado responsable del análisis, planificación, ejecución de consultas e interacción con el cliente.

El proceso del servidor se representa en la figura siguiente.

[p. 3]

[Interpretación de imagen - Figura 1-1: Proceso del servidor PostgreSQL. Diagrama que muestra el proceso Postmaster en el centro, con flechas apuntando a múltiples procesos backend etiquetados como P1, P2, P3. Cada proceso backend (P1/P2/P3) tiene procesos de trabajo en segundo plano correspondientes etiquetados como BW1, BW2, BW3 respectivamente. El diagrama ilustra la relación jerárquica donde el Postmaster genera procesos backend, los cuales a su vez pueden generar trabajadores en segundo plano.]

![**Figura 1-1. Proceso del servidor PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-1.png)

En la Figura 1-1, P1/P2/P3 son los procesos backend, y BW1/BW2/BW3 son los procesos de trabajo en segundo plano correspondientes.

##### **2. Proceso de utilidad**

El proceso de utilidad es una colección de procesos auxiliares. Estos procesos secundarios o suplementarios realizan todas las tareas iniciadas por el proceso del servidor. Aprendamos sobre los diferentes procesos auxiliares.

**Escritor en segundo plano (Background writer):** El escritor en segundo plano maneja escrituras periódicas de búferes sucios desde la memoria compartida hacia el disco. Los *búferes sucios* (*dirty buffers*) son datos modificados que aún no han sido escritos en el disco y pueden perderse si ocurre un reinicio del servidor. Por lo tanto, es esencial vaciar los datos desde los búferes compartidos, y el escritor en segundo plano maneja esto periódicamente, liberando espacio del búfer compartido.

[p. 4]

**Checkpoint:** Este proceso asegura que los puntos de control (*checkpoints*) ocurran periódicamente. Durante un checkpoint, escribe todas las páginas sucias en el disco y vacía todos los datos.

**Escritor WAL (WAL Writer):** El Registro por Adelantado de Escritura (*Write-Ahead Logging*, *WAL*) es un método de registro de transacciones en PostgreSQL que registra los cambios en los archivos de datos en un registro antes de escribirlos en los archivos de datos. El *Escritor WAL* escribe los cambios realizados en los datos en el registro de transacciones para garantizar la durabilidad.

**Autovacuum:** Vacuum son datos que ya no son necesarios. El proceso autovacuum recupera automáticamente el almacenamiento eliminando tuplas muertas. El *iniciador de Autovacuum* (*Autovacuum launcher*) genera procesos autovacuum para limpiar el vacuum y asegurar un rendimiento óptimo.

**Archivador (Archiver):** Responsable de archivar los archivos WAL para soportar la *recuperación en un punto en el tiempo* (*point-in-time recovery*, *PITR*, por sus siglas en inglés). Este proceso respalda todos los registros WAL que son esenciales para PITR y la replicación de datos.

**Recolector de estadísticas (Statistics collector):** Recopila estadísticas sobre el uso de la base de datos y el rendimiento de las consultas, y proporciona información para que el planificador de consultas optimice la ejecución de consultas.

**Escritor de registros (Log writer):** Responsable de escribir registros de depuración y registros de errores en el archivo de registro que son importantes para la solución de problemas y el monitoreo.

**Iniciador de replicación lógica (Logical replication launcher):** Responsable de verificar periódicamente la tabla de catálogo *pg_subscription* para ver si se han agregado o habilitado suscripciones, y asegurar que los trabajadores de replicación lógica se inicien para cada suscripción habilitada, haciendo uso de la infraestructura de trabajadores en segundo plano.

La Figura 1-2 muestra la colección de todos los procesos auxiliares.

[p. 5]

[Interpretación de imagen - Figura 1-2: Proceso de utilidad de PostgreSQL. Diagrama que muestra un concentrador central con flechas conectando a múltiples cajas etiquetadas que representan procesos auxiliares: Escritor en segundo plano, Checkpoint, Escritor WAL, Autovacuum (con iniciador de Autovacuum), Archivador, Recolector de estadísticas, Escritor de registros, e Iniciador de replicación lógica. El diagrama ilustra cómo estos procesos de utilidad operan en paralelo para soportar las operaciones principales del servidor.]

![**Figura 1-2. Proceso de utilidad de PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-2.png)

##### **3. Memoria del sistema**

La arquitectura de memoria de PostgreSQL se divide en dos categorías:

- Memoria local

- Memoria compartida

**Memoria local**

La memoria local es privada para cada proceso en segundo plano utilizado para el procesamiento de consultas. Se divide además en varias subáreas, cuyos tamaños son fijos o variables.

Las subáreas incluyen:

- Memoria de trabajo (Work memory)

- Memoria de trabajo de mantenimiento (Maintenance work memory)

  - Memoria de trabajo de autovacuum (Autovacuum work memory)

[p. 6]

- Búferes temporales (Temporary buffers)

  - Tamaño efectivo de caché (Effective cache size)

  - Caché de catálogo (Catalog cache)

  - Caché del sistema operativo (Operating system cache)

**Memoria de trabajo:** Esta área se utiliza para operaciones de ordenamiento, operaciones de unión (join) y ejecución de consultas.

**Memoria de trabajo de mantenimiento:** Esta área se utiliza para todas las operaciones de mantenimiento, como vacuum, vacuum full, analyze, reindexing, creación de un índice, alteración de una tabla, etc.

**Tamaño efectivo de caché:** Esta área se utiliza para hacer que el uso de índices sea más efectivo.

**Memoria compartida**

La memoria compartida usualmente es asignada por un servidor PostgreSQL cuando se inicia. Esta área se subdivide en varias subáreas de tamaño fijo y es accesible por todos los procesos en segundo plano.

- Búferes compartidos (Shared buffers)

- Búferes WAL (WAL buffers)

- Búferes temporales (Temp buffers)

- Otros búferes (Other buffers)

**Búferes compartidos:** PostgreSQL utiliza esta área para cargar páginas dentro de tablas e índices desde los discos y operar sobre ellas. Por ejemplo, para ver la asignación actual, puede emitir el siguiente comando:

```         
postgres=# show shared_buffers;
 shared_buffers
----------------
128MB
(1 row)
```

[p. 7]

**Búferes WAL:** Esta área almacena entradas del registro de transacciones antes de que sean escritas en el disco para asegurar que las fallas del servidor no hayan causado pérdida de datos. Por ejemplo, para ver la asignación actual, puede emitir el siguiente comando:

```         
postgres=# SHOW wal_buffers;
 wal_buffers
-------------
4MB
(1 row)
```

**Búferes temporales:** Esta área almacena tablas temporales requeridas durante una operación y cuando se necesitan más conjuntos de resultados temporales durante operaciones de unión múltiples. También se utiliza cada vez que la *Memoria de Trabajo* ya no puede soportar la operación de consulta. Por ejemplo, para ver la asignación actual, puede emitir el siguiente comando:

```         
postgres=# SHOW temp_buffers;
 temp_buffers
--------------
8MB
(1 row)
```

**Otros búferes:** Además de estos búferes, PostgreSQL asigna otras áreas para el procesamiento de transacciones, como puntos de guardado (savepoints) y confirmación en dos fases (two-phase commit), procesos en segundo plano, como checkpoints y autovacuum, y diferentes mecanismos de control de acceso, como bloqueos compartidos y exclusivos, etc.

La figura siguiente representa los diversos componentes involucrados en la arquitectura de memoria.

[p. 8]

[Interpretación de imagen - Figura 1-3: Memoria del sistema PostgreSQL. Diagrama dividido en dos secciones principales: Memoria Local y Memoria Compartida. La sección de Memoria Local muestra subáreas: Memoria de trabajo, Memoria de trabajo de mantenimiento, Memoria de trabajo de autovacuum, Búferes temporales, Tamaño efectivo de caché, Caché de catálogo, Caché del sistema operativo. La sección de Memoria Compartida muestra: Búferes compartidos, Búferes WAL, Búferes temporales, Otros búferes. Las flechas indican accesibilidad: la memoria local es privada para cada proceso backend, mientras que la memoria compartida es accesible por todos los procesos en segundo plano.]

![**Figura 1-3. Memoria del sistema PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-3.png)

##### **4. Archivos físicos**

Los archivos físicos son los archivos reales almacenados en el directorio base. Contiene varios subdirectorios diferentes y muchos archivos.

En general, todos los archivos físicos se agrupan en las siguientes categorías:

- Archivos de datos

- Archivos de registro WAL

- Archivos de registro (*log*)

- Archivos de registro archivados

**Archivos de datos**

Los archivos de datos usualmente incluyen los archivos de configuración y los archivos de datos utilizados por el clúster de base de datos y se almacenan en el directorio de datos del clúster, como *PGDATA*. Este directorio contiene además varios [p. 9] subdirectorios, archivos de control y otros archivos de configuración requeridos para ejecutar el clúster de base de datos. Para ver la ubicación predeterminada, emita el siguiente comando:

```         
postgres=# SHOW data_directory;
   data_directory
------------------------
/var/lib/pgsql/16/data
(1 row)

[postgres@pg_server ~]$ ls -lrth /var/lib/pgsql/16/data
```

[Interpretación de imagen - Figura 1-4: Estructura del directorio de datos de PostgreSQL. Diagrama que muestra la estructura del directorio PGDATA con subdirectorios: base/, global/, pg_wal/, pg_log/, archive/, pg_stat/, pg_subtrans/, pg_tblspc/, pg_twophase/, pg_xact/, y archivos de configuración: postgresql.conf, pg_hba.conf, pg_ident.conf, postmaster.opts, postmaster.pid. El diagrama ilustra la organización jerárquica del almacenamiento físico de PostgreSQL.]

![**Figura 1-4. Estructura del directorio de datos de PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-4.png)

[p. 10]

**Archivos de registro WAL**

Los archivos de *Registro por Adelantado de Escritura* (*Write-Ahead Logging*, *WAL*) usualmente se denominan registros de transacciones ya que aseguran la durabilidad de los datos y ayudan a evitar la pérdida de datos en caso de fallas del servidor para la recuperación en un punto en el tiempo y la replicación de datos. La ubicación predeterminada para los archivos de registro WAL es /var/lib/pgsql/16/data/pg_wal.

```         
[postgres@pg_server~]$ ls -lrth /var/lib/pgsql/16/data/pg_wal
total 705M
-rw-------. 1 postgres postgres 16M Abr 28 17:50 00000001000000040000006E
-rw-------. 1 postgres postgres 16M Abr 28 17:50 00000001000000040000006F
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000070
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000071
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000068
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000069
-rw-------. 1 postgres postgres 16M Abr 28 17:50 00000001000000040000006A
-rw-------. 1 postgres postgres 16M Abr 28 17:50 00000001000000040000006B
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000006C
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000006D
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000007D
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000073
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000074
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000075
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000076
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000077
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000078
-rw-------. 1 postgres postgres 16M Abr 28 17:51 000000010000000400000079
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000007A
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000007B
-rw-------. 1 postgres postgres 16M Abr 28 17:51 00000001000000040000007C
-rw-------. 1 postgres postgres 341 May 12 12:07 000000010000000400000066.00000028.backup
drwx------. 2 postgres postgres 59 May 12 12:12 archive_status
-rw-------. 1 postgres postgres 16M May 13 17:17 000000010000000400000067
<<< OUTPUT TRUNCATED >>>
```

[p. 11]

**Archivos de registro (*Log files*)**

Los archivos de registro son donde podemos ver toda la información de diagnóstico requerida para solución de problemas, monitoreo y propósitos analíticos. Cada acción realizada dentro de la base de datos se escribe en el archivo de registro para rastrear todos los cambios dentro de la base de datos. Para ver la ubicación del archivo de registro:

```         
postgres=# SHOW log_directory;
 log_directory
---------------
log
(1 row)

[postgres@pg_server log]$ ls -lrth /var/lib/pgsql/16/data/log
total 96K
-rwx------. 1 postgres postgres 3.5K May  1 17:34 postgresql-Jue.log
-rwx------. 1 postgres postgres 6.9K May  2 22:17 postgresql-Vie.log
-rwx------. 1 postgres postgres  0 May  3 12:10 postgresql-Sáb.log
-rw-------. 1 postgres postgres 19K May  4 12:49 postgresql-Dom.log
-rwx------. 1 postgres postgres 5.0K May  7 21:37 postgresql-Mié.log
-rw-------. 1 postgres postgres 42K May 12 12:12 postgresql-Lun.log
-rwx------. 1 postgres postgres 8.7K May 13 17:23 postgresql-Mar.log
[postgres@pg_server log]$
```

[p. 12]

```         
postgres=# SHOW log_filename;
  log_filename
-------------------
postgresql-%a.log
(1 row)

postgres=# SHOW logging_collector;
 logging_collector
-------------------
on
(1 row)
```

**Archivos de registro archivados**

Los archivos de registro archivados no son más que archivos de registro WAL que han sido movidos a una ubicación de archivo después de haber sido escritos en el disco. Debemos archivar los archivos de registro WAL ya que ayudan a mantener la integridad de los datos, el respaldo y la recuperación, y soportan la replicación. Para ver la ubicación predeterminada:

```         
postgres=# SHOW archive_command;
          archive_command
-----------------------------------------
cp %p /var/lib/pgsql/16/data/archive/%f
(1 row)

[postgres@pg_server~]$ ls -lrth /var/lib/pgsql/16/data/archive/
total 1.7G
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000000
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000001
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000002
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000003
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000004
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000005
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000006
-rw-------. 1 postgres postgres 16M Abr 28 17:50 000000010000000400000007
<<< SALIDA TRUNCADA >>>
```

[p. 13]

La figura siguiente muestra los archivos físicos almacenados en el directorio base.

[Interpretación de imagen - Figura 1-5: Archivos físicos de PostgreSQL. Diagrama que muestra cuatro categorías de archivos físicos: Archivos de datos (con subdirectorios como base/, global/), Archivos de registro WAL (directorio pg_wal/ con archivos de segmento), Archivos de registro (directorio log/ con archivos postgresql-\*.log), y Archivos de registro archivados (directorio archive/ con segmentos WAL archivados). Las flechas indican el flujo de archivos WAL desde pg_wal/ hacia archive/ mediante el proceso archivador.]

![**Figura 1-5. Archivos físicos de PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-5.png)

[p. 14]

#### **Árbol de Procesos del Servidor PostgreSQL**

Ahora que hemos visto cada componente arquitectónico, comprendamos los pasos involucrados en el flujo arquitectónico cuando se inicia una conexión y veamos el árbol de procesos de PostgreSQL.

Un árbol de procesos del servidor PostgreSQL típico se ve como sigue:

[Interpretación de imagen - Figura 1-6: Árbol de procesos del servidor PostgreSQL. Diagrama de árbol jerárquico con Postmaster (postgres -D /data) en la raíz. Las ramas muestran: logger, checkpointer, background writer, walwriter, autovacuum launcher, archiver, logical replication launcher como hijos directos de Postmaster. Ramas adicionales muestran procesos backend (postgres: usuario base_de_datos) generados para conexiones de cliente, cada uno potencialmente con sus propios trabajadores en segundo plano.]

![**Figura 1-6. Árbol de procesos del servidor PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-6.png)

Aquí hay una salida de muestra del árbol de procesos del servidor PostgreSQL:

```         
[postgres@pg_server~]$ ps -ef | grep postgres
postgres  1335     1  0 21:07 ?        00:00:00 /usr/pgsql-16/bin/postgres -D /var/lib/pgsql/16/data/
postgres  1489  1335  0 21:07 ?        00:00:00 postgres: logger
postgres  1537  1335  0 21:07 ?        00:00:00 postgres: checkpointer
postgres  1538  1335  0 21:07 ?        00:00:00 postgres: background writer
postgres  3300  1335  0 21:07 ?        00:00:00 postgres: walwriter
postgres  3301  1335  0 21:07 ?        00:00:00 postgres: autovacuum launcher
postgres  3302  1335  0 21:07 ?        00:00:00 postgres: archiver
postgres  3303  1335  0 21:07 ?        00:00:00 postgres: logical replication launcher
root      4409  4289  0 21:10 pts/0    00:00:00 su - postgres
postgres  4410  4409  0 21:10 pts/0    00:00:00 -bash
postgres  5832  4410  0 21:37 pts/0    00:00:00 ps -ef
postgres  5833  4410  0 21:37 pts/0    00:00:00 grep --color=auto postgres
[postgres@pg_server~]$
```

[p. 15]

```         
[postgres@pg_server~]$ systemctl status postgresql-16
• postgresql-16.service - Servidor de base de datos PostgreSQL 16
   Loaded: loaded (/usr/lib/systemd/system/postgresql-16.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2025-05-07 21:07:44 CDT; 33min ago
     Docs: https://www.postgresql.org/docs/16/static/
  Process: 1299 ExecStartPre=/usr/pgsql-16/bin/postgresql-16-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 1335 (postgres)
    Tasks: 8 (limit: 35703)
   Memory: 39.3M
   CGroup: /system.slice/postgresql-16.service
           ├─1335 /usr/pgsql-16/bin/postgres -D /var/lib/pgsql/16/data/
           ├─1489 postgres: logger
           ├─1537 postgres: checkpointer
           ├─1538 postgres: background writer
           ├─3300 postgres: walwriter
           ├─3301 postgres: autovacuum launcher
           ├─3302 postgres: archiver
           └─3303 postgres: logical replication launcher
[postgres@pg_server~]$
```

**Servidor PostgreSQL:** Esta es una colección de todos los procesos dentro de un clúster de base de datos.

**Proceso del servidor PostgreSQL:** Este es el padre de todos los procesos y se llama *postmaster*. Se inicia al arrancar el servidor y se le asigna un área de memoria compartida en la memoria, inicia varios procesos en segundo plano, inicia procesos asociados con replicación y procesos de trabajo en segundo plano donde sea necesario, y luego espera solicitudes de conexión de clientes.

**Proceso backend:** Este proceso se llama *postgres* y maneja todas las consultas emitidas por un cliente conectado. Cada cliente tiene su proceso backend y sus procesos en segundo plano correspondientes.

[p. 16]

**Proceso en segundo plano:** Este es un grupo de todos los procesos individuales que realizan una acción específica.

**Procesos relacionados con replicación:** Estos procesos realizan todo el procesamiento relacionado con la replicación por streaming.

**Procesos de trabajo en segundo plano:** Estos procesos realizan cualquier procesamiento de usuario, incluyendo código suministrado por el usuario.

#### **Flujo de Arquitectura de PostgreSQL**

En esta sección, combinemos todos los diferentes componentes arquitectónicos que hemos aprendido hasta ahora para comprender cómo ocurre un flujo arquitectónico típico cuando se inicia una solicitud de cliente.

Por favor, siga el flujo observando los números en la Figura 1-7 para comprender mejor el flujo.

1.  Un cliente inicia una solicitud de conexión a la base de datos. Un cliente puede ser cualquier aplicación o interfaz que se comunique con el servidor PostgreSQL para enviar consultas y recibir resultados. Esto puede ser herramientas de línea de comandos (como psql), interfaces gráficas de usuario (GUI), u otras aplicaciones que utilizan bibliotecas de interfaz de cliente de PostgreSQL. Los clientes se conectan al servidor PostgreSQL utilizando un protocolo de red, típicamente TCP/IP. En algunos casos, también se utilizan sockets de dominio Unix para conexiones locales. Cada conexión es autenticada y autorizada en el puerto predeterminado 5432 de PostgreSQL. El proceso del servidor ***postmaster*** escucha continuamente cada nueva conexión entrante.

2.  La conexión ***postmaster*** luego transfiere el procesamiento al proceso backend ***postgres***. Cada conexión de un cliente es manejada por un proceso backend separado, que ejecuta consultas, gestiona transacciones y recupera datos.

3.  Cada proceso backend genera procesos de trabajo en segundo plano separados según sea necesario, dependiendo del tipo de solicitud de cliente recibida. Se comunica con el cliente a través de una única conexión TCP/IP y termina cuando la conexión se desconecta o finaliza.

[p. 17]

4.  Cada proceso que el servidor inicia requiere un componente de memoria para realizar su procesamiento. Por lo tanto, se asigna un área de memoria dependiendo del tipo de procesamiento necesario.

5.  El proceso del servidor también genera procesos auxiliares subsecuentes para realizar diversas acciones en segundo plano necesarias para la gestión de la base de datos, cada una sirviendo un propósito específico.

6.  Después de que todo el procesamiento está completo, dondequiera que los datos confirmados necesiten persistencia, los procesos auxiliares y de memoria correspondientes almacenan los datos en archivos físicos.

[Interpretación de imagen - Figura 1-7: Flujo de arquitectura de PostgreSQL. Diagrama de flujo numerado que muestra: (1) Cliente inicia conexión a Postmaster en el puerto 5432; (2) Postmaster genera proceso Backend para la conexión; (3) Backend genera trabajadores en segundo plano según sea necesario; (4) Áreas de memoria (Búferes compartidos, Memoria de trabajo) asignadas para procesamiento; (5) Procesos de utilidad (Escritor en segundo plano, Escritor WAL, etc.) realizan tareas en segundo plano; (6) Datos confirmados persistidos en Archivos físicos (Archivos de datos, Archivos WAL). Las flechas indican flujo de datos y relaciones de procesos.]

![**Figura 1-7. Flujo de arquitectura de PostgreSQL**](https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/kumar_2025_figure_1-7.png)

[p. 18]

#### Resumen

En resumen, aprendimos sobre los diferentes componentes centrales de la arquitectura de PostgreSQL, sus subcomponentes y su funcionalidad. También aprendimos sobre el flujo de arquitectura basado en procesos de PostgreSQL y vimos cómo se maneja una solicitud de cliente.

Esta arquitectura asegura el manejo eficiente de solicitudes de clientes, la integridad de los datos y un rendimiento robusto en sesiones concurrentes.

------------------------------------------------------------------------

**Nota sobre derechos de autor:** Esta traducción se proporciona exclusivamente con fines educativos y de investigación académica. El contenido original está protegido por derechos de autor © 2025 por Y V Ravi Kumar, Arun Kumar Samayam y Phani Kadambari, publicado por Apress/Springer.
