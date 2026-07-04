### Laboratorio 13. Extensiones en PostgreSQL
(Opcional, pero recomendado)

**Dr. Jesús Zavala Ruiz**  
**Última actualización:** 4 de julio de 2026  

---

#### 1. Objetivos

Al finalizar este laboratorio, el alumno comprenderá la arquitectura de extensibilidad de PostgreSQL y será capaz de gestionar el ciclo de vida de extensiones mediante comandos SQL y operaciones del sistema. Como ejercicio integrador, instalará y configurará PostGIS 3.5 en PostgreSQL 18 para ejecutar operaciones espaciales fundamentales en un entorno controlado.

De manera específica, el estudiante aprenderá a diferenciar entre extensiones `contrib`, comunitarias y corporativas; ejecutará comandos como `CREATE`, `ALTER` y `DROP EXTENSION` comprendiendo sus efectos en el catálogo del sistema; identificará extensiones relevantes según el dominio funcional de su proyecto; se conectará a la máquina virtual mediante SSH con autenticación PKI; y aplicará buenas prácticas de gobernanza como validación de compatibilidad binaria, gestión de versiones y aislamiento de esquemas.

#### 2. Introducción

PostgreSQL no es una caja negra. A diferencia de otros motores de bases de datos que entregan todas sus funcionalidades como un bloque monolítico, PostgreSQL expone interfaces de programación en C que permiten incorporar módulos externos llamados **extensiones**. Una extensión no es un complemento opcional: es un objeto de primera clase que puede definir tipos de datos, funciones, operadores, métodos de acceso a índices o incluso procesos en segundo plano. Todo esto se integra al esquema de destino mediante `CREATE EXTENSION`, de modo que el motor reconoce estos elementos como parte natural del entorno relacional.

Este modelo de extensibilidad responde a principios de diseño claros. La **modularidad** mantiene estable el núcleo del motor: las capacidades especializadas se externalizan para evitar dependencias innecesarias. La **innovación abierta** permite que investigadores, empresas y la comunidad desarrollen funcionalidades avanzadas —geoespaciales, analíticas, de inteligencia artificial— sin modificar el código fuente oficial. Y la convención de nomenclatura `pg_`, aunque frecuente, no tiene significado técnico: es una herencia histórica del directorio `contrib/`. Extensiones tan relevantes como `postgis`, `timescaledb` o `pgvector` no usan este prefijo, y eso no limita su funcionalidad en absoluto.

Para trabajar en este laboratorio, el alumno se conectará a una máquina virtual con Fedora 44 Cloud Base, mediante SSH usando autenticación por clave pública. Las credenciales son unificadas para simplificar el acceso:

- Usuario del SO y PostgreSQL: `alumno`  
- Contraseña: `uamIztapalapa`  
- Clave SSH privada en el host: `~/.ssh/fedora-lab-key`  
- IP de la VM: `192.168.122.24` (variable, según el entorno)  

La máquina virtual fue creada en el [Laboratorio 5](https://github.com/jzavalar/bases-de-datos/blob/main/Lab_05_creacion-de-laboratorio-con-fedora-postgresql-y-pagila.md).

Desde una terminal en el sistema anfitrión, la conexión se establece con:

```bash
ssh -i ~/.ssh/fedora-lab-key alumno@192.168.122.24
```

> **Nota de seguridad:** La conexión no solicita *passphrase* porque es un laboratorio, pero es recomendable protegerla en entornos productivos. Para sesiones prolongadas, considere usar `ssh-agent`.

#### 3. Fundamentos de extensibilidad en PostgreSQL

Las extensiones se clasifican según su origen y nivel de confianza. Las distribuidas con el código fuente oficial (`contrib`) son mantenidas por PostgreSQL Global Development Group e incluyen módulos como `pg_stat_statements`, `pg_trgm` o `hstore`. Estas pueden ser *trusted* (instalables por usuarios con privilegio `CREATE`) o *untrusted* (requieren `SUPERUSER`). Las extensiones comunitarias, publicadas en PGXN, dependen de mantenimiento de terceros pero suelen estar bien documentadas (`pg_partman`, `pg_repack`, `pgaudit`). Finalmente, las extensiones corporativas como `timescaledb`, `citus` o `pgvector` son desarrolladas por empresas y ofrecen licencias abiertas o duales.

La distinción entre *trusted* y *untrusted* es crucial: una extensión *trusted* (confiable) no accede al sistema de archivos ni ejecuta código privilegiado, mientras que una *untrusted* (no confiable) puede requerir inicialización en memoria compartida o interacción con recursos de bajo nivel. PostgreSQL verifica esta propiedad mediante el campo `exttrusted` en el catálogo interno. Esta clasificación determina directamente el modelo de seguridad aplicable en entornos académicos y productivos.

#### 4. Comandos esenciales y ciclo de vida

La gestión de extensiones se realiza mediante un conjunto reducido pero potente de instrucciones SQL. Comprender su funcionamiento interno evita inconsistencias estructurales o degradaciones de rendimiento inadvertidas:

| Acción | Comando SQL | Comentario técnico |
|--------|-------------|-------------------|
| Listar extensiones instaladas | `SELECT * FROM pg_extension;` | Consulta metadatos de versión, esquema y dependencias |
| Instalar una extensión | `CREATE EXTENSION IF NOT EXISTS pg_trgm;` | Registra la extensión y ejecuta scripts de definición en `SHAREDIR/extension/` |
| Actualizar una extensión | `ALTER EXTENSION pg_trgm UPDATE;` | Aplica migraciones versionadas; no requiere reinicio salvo cambios en bibliotecas compartidas |
| Desinstalar una extensión | `DROP EXTENSION IF EXISTS pg_trgm CASCADE;` | Elimina objetos en orden de dependencia inversa; usar `CASCADE` con precaución |
| Carga a nivel de servidor | `shared_preload_libraries = 'pg_stat_statements'` | Requisito para extensiones que reservan memoria compartida o registran hooks en el planner |

Es importante notar que `CREATE EXTENSION` no copia código al directorio de datos: simplemente vincula metadatos del catálogo con bibliotecas compartidas (`.so` en Linux) y scripts SQL de definición. Esta separación permite que PostgreSQL gestione versiones y valide compatibilidad binaria de forma determinista. La directiva `shared_preload_libraries`, por su parte, opera en una fase temprana del ciclo de vida del servidor: la extensión solicita bloques de memoria en `shared_buffers`, registra callbacks en el executor o planner, y en algunos casos lanza procesos auxiliares que sobreviven a reconexiones de clientes. Por eso ciertas extensiones requieren reinicio del servicio para activarse.

Tras un upgrade mayor de PostgreSQL, es imperativo ejecutar `ALTER EXTENSION nombre UPDATE;` para aplicar migraciones de esquema. El motor no actualiza extensiones automáticamente para preservar la integridad transaccional. Además, PostgreSQL resuelve dependencias entre extensiones mediante el catálogo `pg_depend`: si una extensión A requiere B, el sistema impide desinstalar B mientras A esté activa. El mecanismo de resolución de nombres prioriza el esquema `public`, lo cual puede generar colisiones en entornos multiusuario con extensiones instaladas en esquemas personalizados.

#### 5. Catálogo funcional por dominio

A continuación se presentan las extensiones más relevantes para fines académicos, organizadas por dominio de aplicación. Se priorizan aquellas con documentación accesible, estabilidad probada y valor pedagógico.

##### 5.1. Monitoreo y diagnóstico
- `pg_stat_statements` (Contrib, PG 14–18): Estadísticas de ejecución por fingerprint de consulta. Imprescindible para análisis de rendimiento.  
- `auto_explain` (Contrib, PG 14–18): Registro automático de planes de ejecución para consultas lentas. Útil para estudiar el optimizador.  
- `pg_wait_sampling` (Contrib, PG 14–18): Muestreo de eventos de espera (locks, I/O). Complementario para diagnóstico de contención.  

##### 5.2. Seguridad y auditoría
- `pgaudit` (Externa, PG 14–18): Auditoría granular de operaciones DDL/DML. Estándar para cumplimiento normativo.  
- `passwordcheck` (Contrib, PG 14–18): Validación de fortaleza de contraseñas. Simple y pedagógico.  
- `pgsodium` (Externa, PG 14–17): Criptografía moderna con libsodium. Alternativa recomendada a `pgcrypto` (obsoleto desde PG 14).  

##### 5.3. Tipos de datos avanzados
- `postgis` (Externa, PG 14–18): Tipos espaciales, índices GIS. Referente industrial para modelado geoespacial.  
- `hstore` (Contrib, PG 14–18): Almacenamiento de pares clave-valor. Introducción a estructuras semiestructuradas en SQL.  
- `citext` (Contrib, PG 14–18): Texto con comparación *case-insensitive*. Evita errores frecuentes en búsquedas de identidad.  
- `ltree` (Contrib, PG 14–18): Representación de jerarquías. Útil para categorías, ACLs y árboles de decisión.  

##### 5.4. Búsqueda de texto y lingüística
- `pg_trgm` (Contrib, PG 14–18): Índice de trigramas para búsquedas difusas. Fundamental para estudiar métodos de acceso GiST/GIN.  
- `unaccent` (Contrib, PG 14–18): Eliminación de diacríticos. Combinar con `pg_trgm` para soporte multilingüe.  
- `fuzzystrmatch` (Contrib, PG 14–18): Algoritmos Soundex, Metaphone, Levenshtein. Pedagogía en coincidencia aproximada.  

##### 5.5. Gestión de almacenamiento y mantenimiento
- `pg_partman` (Externa, PG 14–18): Automatización de particionamiento nativo. Excelente para estudiar escalabilidad horizontal.  
- `pg_repack` (Externa, PG 14–17): Reorganización en línea sin bloqueo exclusivo. Requiere acceso local; no compatible con cloud serverless.  
- `pg_buffercache` (Contrib, PG 14–18): Inspección de la caché de buffers. Útil para entender `shared_buffers` y patrones de E/S.  

##### 5.6. Series temporales y analítica
- `timescaledb` (Externa, PG 14–18): Hipertablas, compresión y agregados continuos. Referente en IoT y métricas.  
- `pg_ivm` (Contrib, PG 17–18): Vistas materializadas incrementales sin triggers. Concepto moderno de optimización analítica.  
- `postgres_fdw` (Contrib, PG 14–18): Acceso federado a bases remotas. Base para estudiar federación y migraciones.  

##### 5.7. Inteligencia artificial y vectores
- `pgvector` (Externa, PG 14–18): Tipo `vector` e índices HNSW/IVFFlat para embeddings. Estándar actual para RAG y búsqueda semántica.  
- `pgvectorscale` (Externa, PG 15–18): Optimización de almacenamiento para vectores de alta dimensión. Complementario para rendimiento a escala.  
- `pgai` (Externa, PG 16–18): Orquestación de llamadas a LLMs desde SQL. Emergente; útil para integración IA-DB.  

Cada dominio explota mecanismos internos distintos del motor. Las extensiones de búsqueda de texto usan métodos de acceso GiST/GIN que indexan fragmentos en lugar de valores completos. Las soluciones de series temporales reescriben estrategias de particionamiento para optimizar compresión e ingesta secuencial. Los módulos de IA introducen estructuras de datos en memoria y algoritmos de aproximación de vecinos que coexisten con el planificador tradicional. Esta diversidad demuestra que PostgreSQL permite intervención directa en el motor de almacenamiento, el optimizador y el ejecutor, no solo en la capa de aplicación.

#### 6. Estudio de caso: PostGIS

##### 6.1. Contexto y preparación del entorno

Este ejercicio se ejecuta en una máquina virtual con Fedora 44, con credenciales unificadas para el sistema operativo y PostgreSQL. PostGIS es la extensión de referencia industrial para datos geoespaciales: permite almacenar, indexar y consultar geometrías mediante funciones especializadas, operadores espaciales y métodos de acceso GIST (Geographic Information Science and Technology o Ciencia y Tecnología de la Información Geográfica). En otras palabras, PostGIS convierte a PostgreSQL en un Sistema de Información Geográfica (Geographical Information System, GIS) y lo habilita para manejar [bases de datos espaciales](https://postgis.net/workshops/postgis-intro/introduction.html). 

##### 6.2. Instalación de PostgreSQL y PostGIS

Desde la terminal del usuario `alumno`, actualice el sistema y configure el repositorio oficial de PostgreSQL:

```bash
# Instalar el repo EPEL:
sudo dnf -y install epel-release

# Habilitar el repo PowerTools (requerudo por algunas de las dependencias):
sudo dnf -y config-manager --set-enabled PowerTools

# Para Rocky Linux necesita instalar esto en lugar de lo anterior
sudo dnf config-manager --enable crb

# Para Rocky 8+ necesita tambi'en habilitar esto
sudo crb enable

# Ahora, finalmente, instale PostGIS
# Elija las versiones correctas de PostGIS y PostgreSQL
sudo dnf -y install postgis35_18

# Reinicie postgres
sudo systemctl restart postgresql
```

El paquete `postgis35_18` sigue la convención de Fedora: `postgis<versión>_<versión-pg>`. La biblioteca `postgis-3.5.so` se instala en `/usr/pgsql-18/lib/`, ruta que PostgreSQL consulta automáticamente. Inicialice el clúster y habilite el servicio:

```bash
sudo postgresql-setup --initdb
sudo systemctl enable postgresql.service
sudo systemctl start postgresql.service
```

En Fedora, el servicio se registra como `postgresql-18` para permitir coexistencia de versiones. La herramienta de configuración establece autenticación `peer` para usuarios locales, facilitando el acceso sin contraseña para el rol `postgres`.

La instrucción `module disable postgresql` evita conflictos con el módulo predeterminado de Fedora, que podría ofrecer una versión incompatible. Luego instale los binarios del motor y la extensión geoespacial con sus dependencias (PROJ, GEOS, GDAL):

##### 6.3. Configuración de roles y activación de PostGIS

Cree el rol académico con privilegios controlados:

```bash
sudo -u postgres psql -c "CREATE ROLE alumno WITH LOGIN PASSWORD 'uamIztapalapa' CREATEDB;"
sudo -u postgres psql -c "CREATE DATABASE gis_lab OWNER alumno;"
```

El privilegio `CREATEDB` permite al usuario gestionar sus propias bases de datos sin intervención del administrador, siguiendo el principio de menor privilegio. Conéctese a la base de datos y active la extensión:

```bash
psql -U alumno -d gis_lab -h localhost
```

Dentro de `psql`:

```sql
\c gis_lab
CREATE EXTENSION postgis;
SELECT PostGIS_Version();
```

La instrucción `CREATE EXTENSION postgis` ejecuta el script `postgis--3.5.sql`, que registra más de 1.200 funciones, tipos espaciales (`geometry`, `geography`), operadores y vistas de sistema. La función `PostGIS_Version()` confirma que la biblioteca se cargó correctamente.

##### 6.4. Ejercicio práctico: operaciones espaciales fundamentales

Defina una tabla con columna geométrica usando el sistema de referencia WGS 84 (SRID 4326):

```sql
CREATE TABLE campus (
    id serial PRIMARY KEY,
    nombre text NOT NULL,
    coordenadas geometry(Point, 4326)
);
```

La declaración `geometry(Point, 4326)` aplica constraints automáticos que validan dimensionalidad y sistema de referencia antes de escribir en WAL, previniendo inconsistencias topológicas tempranas. Inserte coordenadas de unidades académicas:

```sql
INSERT INTO campus (nombre, coordenadas) VALUES 
    ('UAM Iztapalapa', ST_SetSRID(ST_MakePoint(-99.0766, 19.3610), 4326)),
    ('UAM Azcapotzalco', ST_SetSRID(ST_MakePoint(-99.1883, 19.4901), 4326)),
    ('UAM Xochimilco', ST_SetSRID(ST_MakePoint(-99.1033, 19.2897), 4326));
```

Cree un índice GIST para optimizar consultas de proximidad:

```sql
CREATE INDEX idx_campus_coordenadas ON campus USING GIST (coordenadas);
```

Los índices GIST organizan geometrías en cuadros delimitadores jerárquicos, reduciendo la complejidad de búsqueda espacial de O(n) a O(log n). Sin este índice, operaciones como `ST_DWithin` ejecutarían escaneos secuenciales completos.

Calcule la distancia métrica entre campus usando el tipo `geography` para precisión elipsoidal:

```sql
SELECT 
    origen.nombre AS campus_origen,
    destino.nombre AS campus_destino,
    ST_Distance(
        origen.coordenadas::geography, 
        destino.coordenadas::geography
    ) AS distancia_metros
FROM campus origen, campus destino
WHERE origen.nombre = 'UAM Iztapalapa' 
  AND destino.nombre = 'UAM Azcapotzalco';
```

El casting `::geography` transforma coordenadas en un modelo elipsoidal WGS 84 que calcula distancias geodésicas precisas en metros. El tipo `geometry` opera en espacio plano cartesiano; su uso para mediciones de larga distancia introduce distorsiones significativas. Esta dualidad es fundamental en sistemas de información geográfica.

Finalmente, identifique campus dentro de un radio de 15 kilómetros:

```sql
SELECT 
    nombre,
    ST_AsText(coordenadas) AS wkt,
    ST_Distance(
        coordenadas::geography,
        ST_SetSRID(ST_MakePoint(-99.0766, 19.3610), 4326)::geography
    ) AS distancia_desde_iztapalapa_metros
FROM campus
WHERE ST_DWithin(
    coordenadas::geography,
    ST_SetSRID(ST_MakePoint(-99.0766, 19.3610), 4326)::geography,
    15000
)
ORDER BY distancia_desde_iztapalapa_metros;
```

`ST_DWithin` aprovecha el índice GIST para filtrar candidatos mediante comparación de bounding boxes antes de aplicar el cálculo geodésico exacto. Este patrón de "filtrado grueso + refinamiento fino" es estándar en consultas espaciales de alto rendimiento.

##### 6.5. Validación y cierre del laboratorio

Verifique el estado de la extensión consultando el catálogo:

```sql
SELECT extname, extversion, nspname 
FROM pg_extension e
JOIN pg_namespace n ON e.extnamespace = n.oid
WHERE extname = 'postgis';
```

Al finalizar, elimine la base de datos para liberar recursos:

```bash
psql -U alumno -d postgres -h localhost -c "DROP DATABASE gis_lab;"
```

La eliminación revoca automáticamente todos los objetos asociados; no se requiere `DROP EXTENSION` explícito.

##### 6.6. Consideraciones para entornos productivos 

PostGIS y PostgreSQL siguen ciclos de actualización independientes; verifique compatibilidad antes de actualizar. En entornos multiusuario, instale PostGIS en un esquema dedicado para evitar colisiones de nombres. Para cargas masivas (más de 10⁵ geometrías), deshabilite temporalmente `full_page_writes` y use `COPY` con transacciones agrupadas para reducir sobrecarga de WAL. Emplee `ST_IsValid()` en triggers de inserción para rechazar geometrías topológicamente inconsistentes.

Para profundizar, consulte las fuentes oficiales:

- Guía de inicio: <https://postgis.net/documentation/getting_started/>  
- Taller introductorio: <https://postgis.net/workshops/postgis-intro/index.html>  
- Manual técnico v3.6: <https://postgis.net/docs/manual-3.6/>  

##### 6.7. Recursos complementarios 

Complete el taller interactivo "PostGIS Introduction" para practicar carga de archivos *Shapefiles* (.shp) y su conversión a base de datos y la transformación de coordenadas. Consulte la sección "Spatial Reference Systems" del manual oficial para comprender gestión del SRID (*Spatial Reference System Identifier* o Identificador del Sistema de Referencia Espacial). Experimente con `ST_AsGeoJSON()` para integrar resultados espaciales con aplicaciones web modernas. Si le llama la atención el mundo de la geografía y los sistemas de información geográfica (SIG o GIS) comience por aprender cartografía con los tres volúmenes de *Basic Cartography for students and technicians* de Anson y Ormeling (1993, 2002, 1996) y el software de la [Open Source Geospatial Foundation](https://www.osgeo.org/projects/).

#### 7. Conclusiones

Este laboratorio ha permitido explorar la arquitectura de extensibilidad que distingue a PostgreSQL de otros sistemas de gestión de bases de datos relacionales. A lo largo de la sesión, el alumno ha transitado desde los fundamentos conceptuales —comprendiendo que una extensión es un objeto de primera clase, no un complemento accesorio— hasta la implementación práctica de PostGIS en un entorno controlado con Fedora 44 y PostgreSQL 18.

La capacidad de PostgreSQL para integrar funcionalidades especializadas sin modificar su núcleo relacional representa una ventaja estratégica tanto para fines académicos como productivos. El estudiante ha aprendido a clasificar extensiones según su origen y nivel de confianza, a gestionar su ciclo de vida mediante comandos SQL (`CREATE`, `ALTER`, `DROP EXTENSION`) y a evaluar su aplicabilidad según el dominio funcional del proyecto. El ejercicio con PostGIS consolidó estos conceptos al poner en práctica operaciones espaciales fundamentales: creación de geometrías con referencia espacial, indexación GIST para optimización de consultas y cálculo de distancias geodésicas mediante el tipo `geography`.

Más allá de la ejecución técnica, este laboratorio invita a reflexionar sobre el papel de la extensibilidad en el diseño de sistemas manejadores de datos modernos. PostgreSQL no impone una visión única de lo que debe ser una base de datos; en cambio, proporciona mecanismos para que la comunidad, la academia y la industria definan esas visiones mediante módulos especializados. Esta filosofía de innovación abierta es la que ha permitido la emergencia de extensiones como `pgvector` para inteligencia artificial, `timescaledb` para series temporales o `citus` para distribución horizontal, posicionando a PostgreSQL como una plataforma multipropósito capaz de adaptarse a desafíos emergentes.

Como próximos pasos, se recomienda al estudiante:

- Explorar otras extensiones del catálogo según intereses de investigación o proyectos personales.  
- Profundizar en la documentación oficial de PostGIS para dominar transformaciones de coordenadas, análisis topológico y publicación de datos espaciales vía servicios web.  
- Experimentar con la combinación de extensiones (por ejemplo, `pgvector` + `pg_trgm` para búsqueda híbrida semántica-lexical) y evaluar sinérgias funcionales.  
- Documentar hallazgos y dificultades en bitácoras de laboratorio, fomentando la práctica reflexiva y la construcción colaborativa de conocimiento.  

La competencia en administración de extensiones no se limita a ejecutar comandos: implica comprender los sacrificios o compensaciones en el rendimiento, gestionar dependencias binarias, aplicar principios de seguridad y anticipar impactos en mantenimiento a largo plazo. Estas habilidades, desarrolladas mediante práctica guiada y consulta crítica de fuentes técnicas, constituyen la base para el diseño de arquitecturas de datos robustas, escalables y alineadas con los requerimientos de contextos institucionales y profesionales.

#### 8. Referencias

Anson, R. W., & Ormeling, F. J. (Eds.). (1993). *Basic cartography for students and technicians Vol. 1* (2nd ed.). Pergamon Press.  
Anson, R. W., & Ormeling, F. J. (Eds.). (2002). *Basic cartography for students and technicians Vol. 2* (2nd ed.). Butterworth-Heinemann.  
Anson, R. W., & Ormeling, F. J. (Eds.). (1996). *Basic cartography for students and technicians Vol. 3*. Butterworth-Heinemann.  
Deprez, D. (2021). *The art of PostgreSQL*. Manning Publications.  
Obe, R. O., & Hsu, L. S. (2021). *PostGIS in action* (3rd. ed.). Manning Publications.  
OSGeo. (2026). Open Source Geospatial Foundation [Computer software]. <https://www.osgeo.org/projects/>
PostGIS Project Steering Committee. (2026). *PostGIS getting started guide*. <https://postgis.net/documentation/getting_started/>  
PostGIS Project Steering Committee. (2026). *Introduction to PostGIS*. PostGIS Workshop. <https://postgis.net/workshops/postgis-intro/index.html>  
PostGIS Project Steering Committee. (2026). *PostGIS 3.6 manual*. <https://postgis.net/docs/manual-3.6/>  
PostGIS Project Steering Committee. (2026). *PostGIS* [Computer software]. GitHub. <https://github.com/postgis/postgis>  
PostgreSQL Extension Network. (2026). *PGXN: PostgreSQL Extension Network*. <https://pgxn.org/>  
PostgreSQL Global Development Group. (2026). *Additional supplied modules*. PostgreSQL Documentation. <https://www.postgresql.org/docs/current/contrib.html>  
PostgreSQL Global Development Group. (2026). *Contrib modules* [Computer software]. GitHub. <https://github.com/postgres/postgres/tree/master/contrib>  
Worsley, J., & Drake, J. (2016). *PostgreSQL: Up and running* (3rd. ed.). O'Reilly Media.  
 
