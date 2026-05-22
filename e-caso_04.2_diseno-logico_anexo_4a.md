# Anexo 4.A: Documento de Diseño Lógico Detallado Completo
## Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas

**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Licenciatura:** Computación  
**UEAs:** Bases de Datos (2151106), Análisis y Diseño de Sistemas, Ingeniería de Software  
**Autor:** Dr. Jesús Zavala Ruiz  
**Última Actualización:** 22 de mayo de 2026  
**Vinculación:** Complemento técnico de la Parte 4: Diseño Lógico  

---

### 4.A.1. Propósito y Alcance del Anexo

Si el diseño conceptual permitió comprender la naturaleza operativa del negocio y la especificación de requisitos estableció las capacidades necesarias del sistema, este anexo define la estructura formal de los datos. Aquí no se prescriben motores de almacenamiento, lenguajes de consulta ni plataformas de despliegue; se documentan entidades, atributos, relaciones y reglas de integridad que deben satisfacer principios matemáticos y operativos antes de su materialización técnica.

Este documento consolida la especificación técnica completa del diseño lógico. Su función es actuar como plano arquitectónico para la fase de diseño físico, garantizando que cada decisión de modelado esté documentada, verificable y trazable. Al finalizar este anexo, se habrá transformado un dominio de negocio analógico en un modelo relacional robusto, preparado para su implementación sin ambigüedades.

**Alcance:**
- Especificación exhaustiva de las quince entidades del modelo lógico (trece nucleares y dos asociativas).
- Matriz de relaciones con cardinalidades, participación y mecanismos de resolución.
- Catálogo estructurado de restricciones de integridad (dominio, referencial, unicidad y nulidad).
- Definición conceptual de vistas lógicas para soporte operativo y analítico.
- Validación formal de cumplimiento de formas normales (1FN, 2FN y 3FN).
- Evidencia empírica de normalización con patrones de datos reales.
- Matriz de trazabilidad bidireccional entre requisitos y estructura lógica.

### 4.A.2. Convenciones de Representación Lógica y Fundamento del Modelo Relacional

Antes de examinar las estructuras, es indispensable establecer las convenciones de notación y los principios que rigen esta fase del diseño.

**Decisión de Adoptar el Modelo Relacional:**  
Para este estudio de caso se ha seleccionado formalmente el modelo relacional como paradigma de diseño lógico. Esta elección responde a su capacidad demostrada para garantizar integridad referencial mediante restricciones declarativas, eliminar redundancias mediante normalización matemática y soportar consultas complejas mediante álgebra relacional. Aunque el diseño físico posterior podría materializarse en diversos entornos tecnológicos, la base relacional asegura portabilidad, auditabilidad y escalabilidad sin comprometer la semántica del negocio.

| Convención | Especificación | Justificación Técnica |
|------------|---------------|----------------------|
| **Idioma** | Inglés para nombres de entidades y atributos | Estándar internacional en ingeniería de datos. Facilita la interoperabilidad, la documentación técnica y la integración con herramientas de modelado. |
| **Nomenclatura** | `snake_case` en minúsculas (`rental_date`, `manager_staff_id`) | Separa visualmente los componentes semánticos, evita conflictos con palabras reservadas y garantiza portabilidad entre entornos sensibles a mayúsculas. |
| **Identificadores** | Sufijo `_id` en claves primarias y foráneas | Establece un patrón reconocible que facilita la navegación del modelo, la construcción de relaciones y la auditoría de integridad. |
| **Tipos de Dato Abstractos** | `IDENTIFIER`, `STRING(n)`, `TEXT`, `INTEGER`, `DECIMAL(p,s)`, `DATE`, `DATETIME`, `BOOLEAN`, `BLOB`, `COLLECTION(T)`, `DERIVED` | Mantienen la independencia tecnológica. La semántica del dato se preserva independientemente del motor subyacente. |

**Nota pedagógica:** Un error frecuente consiste en confundir el nivel lógico con el físico. En esta fase, `DATETIME` indica únicamente la necesidad de registrar fecha y hora combinadas; la elección del tipo de almacenamiento, la gestión de zonas horarias o el motor subyacente corresponden exclusivamente a la fase de diseño físico.

### 4.A.3. Especificación Detallada de Entidades y Atributos

A continuación se presenta la estructura lógica de cada entidad. Para garantizar precisión, formalismo y claridad didáctica, se incluye la columna **Dominio**, que consigna formalmente el conjunto de valores permitidos, y la columna **Ejemplo**, que ilustra instancias concretas de datos.

#### Catálogo Maestro de Contenidos

**Entidad: `actor`**  

Propósito: Representar a las personas naturales que intervienen en producciones audiovisuales. Esta entidad captura la identidad profesional del elenco, independientemente de las películas en las que haya participado.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `actor_id` | `IDENTIFIER` | Identificador invariante del actor | Números enteros positivos generados por secuencia | `42` | `PK`, `NOT NULL` |
| `first_name` | `STRING(45)` | Nombre de pila o artístico | Cualquier cadena alfanumérica de 1 a 45 caracteres | `Keanu` | `NOT NULL` |
| `last_name` | `STRING(45)` | Apellido o segundo nombre artístico | Cualquier cadena alfanumérica de 1 a 45 caracteres | `Reeves` | `NOT NULL` |
| `last_update` | `DATETIME` | Instante de la última modificación del registro | Fecha y hora válida (formato ISO 8601) | `2026-05-22 10:15:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `film`**  

Propósito: Constituir el núcleo del catálogo. Cada registro representa un título audiovisual disponible para renta, con sus metadatos comerciales, técnicos y de clasificación.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `film_id` | `IDENTIFIER` | Identificador único del título | Números enteros positivos generados por secuencia | `105` | `PK`, `NOT NULL` |
| `title` | `STRING(255)` | Nombre comercial con el que se exhibe | Cualquier cadena alfanumérica de 1 a 255 caracteres | `The Matrix` | `NOT NULL` |
| `description` | `TEXT` | Sinopsis o notas de catalogación | Cadena de texto de longitud variable ilimitada | `A computer hacker learns about the true nature of reality...` | `NULL` |
| `release_year` | `INTEGER` | Año de estreno comercial | Entero entre 1900 y año_actual + 5 | `1999` | `NULL`, `CHECK (release_year BETWEEN 1900 AND EXTRACT(YEAR FROM CURRENT_DATE) + 5)` |
| `language_id` | `IDENTIFIER` | Idioma de la versión disponible | Referencia válida a `language.language_id` | `1` | `FK → language`, `NOT NULL` |
| `original_language_id` | `IDENTIFIER` | Idioma de producción original | Referencia válida a `language.language_id` o nulo | `3` | `FK → language`, `NULL` |
| `rental_duration` | `INTEGER` | Días permitidos por contrato de renta | Entero positivo | `3` | `DEFAULT 3`, `CHECK (rental_duration > 0)` |
| `rental_rate` | `DECIMAL(4,2)` | Tarifa base por período | Número decimal con dos cifras decimales, mayor o igual a cero | `4.99` | `DEFAULT 0.00`, `CHECK (rental_rate >= 0)` |
| `length` | `INTEGER` | Duración en minutos | Entero positivo | `136` | `NULL`, `CHECK (length > 0)` |
| `replacement_cost` | `DECIMAL(5,2)` | Valor comercial ante pérdida | Número decimal con dos cifras decimales, mayor o igual a cero | `20.99` | `DEFAULT 0.00`, `CHECK (replacement_cost >= 0)` |
| `rating` | `STRING(10)` | Clasificación etaria normativa | Conjunto cerrado: `{'G', 'PG', 'PG-13', 'R', 'NC-17'}` | `PG-13` | `DEFAULT 'G'`, `CHECK (rating IN ('G','PG','PG-13','R','NC-17'))` |
| `special_features` | `COLLECTION(STRING)` | Características técnicas o de edición | Arreglo de cadenas con valores predefinidos (Trailers, Commentaries, Deleted Scenes, Behind the Scenes) | `{Trailers, Behind the Scenes}` | `NULL` |
| `fulltext` | `DERIVED` | Vector para búsqueda semántica | Generado automáticamente a partir de `title` y `description` | `'matrix':1 'hacker':4 'reality':7` | `NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-22 09:00:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `category`**  

Propósito: Clasificar títulos por afinidad temática, género o formato comercial. Funciona como catálogo maestro independiente para evitar inconsistencias en la etiqueta temática.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `category_id` | `IDENTIFIER` | Identificador único de la categoría | Números enteros positivos generados por secuencia | `6` | `PK`, `NOT NULL` |
| `name` | `STRING(25)` | Denominación temática | Cadena alfanumérica única de 1 a 25 caracteres | `Horror` | `NOT NULL`, `UNIQUE` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-20 12:00:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `language`**  

Propósito: Enumerar los idiomas soportados en el catálogo, tanto para versiones comerciales como para idiomas originales de producción.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `language_id` | `IDENTIFIER` | Identificador único del idioma | Números enteros positivos generados por secuencia | `2` | `PK`, `NOT NULL` |
| `name` | `STRING(20)` | Denominación oficial | Cadena alfanumérica única de 1 a 20 caracteres | `Spanish` | `NOT NULL`, `UNIQUE` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-18 08:30:00` | `NOT NULL`, `DEFAULT NOW()` |

#### Infraestructura Geográfica y Organizacional

**Entidad: `country`**  

Propósito: Normalizar la referencia geográfica de primer nivel. Centraliza países para que un cambio en la denominación oficial se propague automáticamente.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `country_id` | `IDENTIFIER` | Identificador único del país | Números enteros positivos generados por secuencia | `25` | `PK`, `NOT NULL` |
| `country` | `STRING(50)` | Nombre oficial o común | Cadena alfanumérica única de 1 a 50 caracteres | `Mexico` | `NOT NULL`, `UNIQUE` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-10 11:00:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `city`**  

Propósito: Desagregar la geografía al nivel urbano. Cada ciudad se vincula estrictamente a un país, garantizando coherencia jerárquica.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `city_id` | `IDENTIFIER` | Identificador único de la ciudad | Números enteros positivos generados por secuencia | `312` | `PK`, `NOT NULL` |
| `city` | `STRING(50)` | Nombre oficial de la ciudad | Cadena alfanumérica de 1 a 50 caracteres | `Ciudad de México` | `NOT NULL` |
| `country_id` | `IDENTIFIER` | País de pertenencia | Referencia válida a `country.country_id` | `25` | `FK → country`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-10 11:05:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `address`**  

Propósito: Almacenar ubicaciones físicas normalizadas. Es un activo compartido: clientes, empleados y tiendas pueden apuntar al mismo registro si coinciden en ubicación, eliminando redundancia.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `address_id` | `IDENTIFIER` | Identificador único de la dirección | Números enteros positivos generados por secuencia | `158` | `PK`, `NOT NULL` |
| `address` | `STRING(50)` | Calle y número exterior | Cadena alfanumérica de 1 a 50 caracteres | `Av. Insurgentes Sur 1234` | `NOT NULL` |
| `address2` | `STRING(50)` | Complemento (depto, oficina, colonia) | Cadena alfanumérica de 0 a 50 caracteres | `Col. Del Valle, Int. 4B` | `NULL` |
| `district` | `STRING(20)` | Delegación, municipio o zona | Cadena alfanumérica de 1 a 20 caracteres | `Benito Juárez` | `NOT NULL` |
| `city_id` | `IDENTIFIER` | Ciudad de referencia | Referencia válida a `city.city_id` | `312` | `FK → city`, `NOT NULL` |
| `postal_code` | `STRING(10)` | Código postal normalizado | Cadena alfanumérica de 0 a 10 caracteres | `03100` | `NULL` |
| `phone` | `STRING(20)` | Teléfono de contacto directo | Formato numérico internacional válido (E.164) | `+52 55 1234 5678` | `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-22 14:00:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `store`**  

Propósito: Representar un establecimiento físico de renta. Cada tienda tiene ubicación única y un administrador designado, quien no puede ejercer esa función en otra sede simultáneamente.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `store_id` | `IDENTIFIER` | Identificador único de la tienda | Números enteros positivos generados por secuencia | `2` | `PK`, `NOT NULL` |
| `manager_staff_id` | `IDENTIFIER` | Empleado responsable de la sede | Referencia válida y única a `staff.staff_id` | `14` | `FK → staff`, `NOT NULL`, `UNIQUE` |
| `address_id` | `IDENTIFIER` | Ubicación física del local | Referencia válida a `address.address_id` | `158` | `FK → address`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-21 16:00:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `staff`**  

Propósito: Modelar al personal operativo. Registra identidad, credenciales de acceso, adscripción laboral y estado activo. Se diferencia al administrador mediante la relación en `store`.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `staff_id` | `IDENTIFIER` | Identificador único del empleado | Números enteros positivos generados por secuencia | `14` | `PK`, `NOT NULL` |
| `first_name` | `STRING(45)` | Nombre de pila | Cadena alfanumérica de 1 a 45 caracteres | `María` | `NOT NULL` |
| `last_name` | `STRING(45)` | Apellido | Cadena alfanumérica de 1 a 45 caracteres | `García` | `NOT NULL` |
| `address_id` | `IDENTIFIER` | Domicilio particular | Referencia válida a `address.address_id` | `158` | `FK → address`, `NOT NULL` |
| `picture` | `BLOB` | Fotografía para identificación | Secuencia de bytes binarios | `[datos_binarios]` | `NULL` |
| `email` | `STRING(50)` | Correo corporativo o personal | Formato RFC 5322 válido o nulo | `maria.garcia@pagila.com` | `NULL` |
| `store_id` | `IDENTIFIER` | Tienda de adscripción | Referencia válida a `store.store_id` | `2` | `FK → store`, `NOT NULL` |
| `active` | `BOOLEAN` | Estado laboral vigente | `{TRUE, FALSE}` | `TRUE` | `DEFAULT TRUE` |
| `username` | `STRING(16)` | Credencial de acceso al sistema | Cadena alfanumérica única de 1 a 16 caracteres | `mgarcia` | `NOT NULL`, `UNIQUE` |
| `password` | `STRING(40)` | Hash criptográfico de acceso | Cadena hexadecimal de 40 caracteres (SHA-1) | `5baa61e4c9b93f3f0682250b6cf8331b...` | `NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-22 08:00:00` | `NOT NULL`, `DEFAULT NOW()` |

#### Transacciones y Relación con el Cliente

**Entidad: `customer`**  

Propósito: Representar a los usuarios finales del servicio. Se registra en una tienda de origen, conserva su historial aunque cambie su estado operativo y mantiene dirección de contacto.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `customer_id` | `IDENTIFIER` | Identificador único del cliente | Números enteros positivos generados por secuencia | `301` | `PK`, `NOT NULL` |
| `store_id` | `IDENTIFIER` | Tienda donde se registró | Referencia válida a `store.store_id` | `2` | `FK → store`, `NOT NULL` |
| `first_name` | `STRING(45)` | Nombre de pila | Cadena alfanumérica de 1 a 45 caracteres | `Carlos` | `NOT NULL` |
| `last_name` | `STRING(45)` | Apellido | Cadena alfanumérica de 1 a 45 caracteres | `Mendoza` | `NOT NULL` |
| `email` | `STRING(50)` | Contacto electrónico | Formato RFC 5322 válido o nulo | `carlos.m@email.com` | `NULL` |
| `address_id` | `IDENTIFIER` | Domicilio de facturación o envío | Referencia válida a `address.address_id` | `158` | `FK → address`, `NOT NULL` |
| `activebool` | `BOOLEAN` | Estado de la cuenta para rentas | `{TRUE, FALSE}` | `TRUE` | `DEFAULT TRUE` |
| `create_date` | `DATE` | Fecha de alta en el sistema | Fecha válida (formato ISO 8601) | `2024-11-10` | `DEFAULT CURRENT_DATE` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-22 12:30:00` | `NOT NULL` |
| `active` | `INTEGER` | Código alternativo de estado (legacy) | `{0, 1}` o nulo | `1` | `NULL` |

**Entidad: `inventory`**  

Propósito: Materializar el concepto de copia física distinguible. Cada ejemplar tiene estado propio y pertenece a una tienda específica.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `inventory_id` | `IDENTIFIER` | Identificador único del ejemplar | Números enteros positivos generados por secuencia | `4021` | `PK`, `NOT NULL` |
| `film_id` | `IDENTIFIER` | Título al que pertenece | Referencia válida a `film.film_id` | `105` | `FK → film`, `NOT NULL` |
| `store_id` | `IDENTIFIER` | Sede donde se encuentra | Referencia válida a `store.store_id` | `2` | `FK → store`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-22 15:10:00` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `rental`**  

Propósito: Capturar la transacción nuclear del negocio. Vincula cliente, ejemplar y empleado en un instante temporal preciso, registrando también la devolución cuando ocurre.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `rental_id` | `IDENTIFIER` | Identificador único de la renta | Números enteros positivos generados por secuencia | `10402` | `PK`, `NOT NULL` |
| `rental_date` | `DATETIME` | Fecha y hora exacta de salida | Fecha y hora válida (formato ISO 8601) | `2026-05-20 18:45:00` | `NOT NULL` |
| `inventory_id` | `IDENTIFIER` | Ejemplar que sale de tienda | Referencia válida a `inventory.inventory_id` | `4021` | `FK → inventory`, `NOT NULL` |
| `customer_id` | `IDENTIFIER` | Cliente que lo toma | Referencia válida a `customer.customer_id` | `301` | `FK → customer`, `NOT NULL` |
| `return_date` | `DATETIME` | Fecha y hora de devolución | Fecha y hora válida posterior a `rental_date`, o nulo | `2026-05-23 14:00:00` | `NULL` |
| `staff_id` | `IDENTIFIER` | Empleado que procesó la operación | Referencia válida a `staff.staff_id` | `14` | `FK → staff`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-23 14:00:05` | `NOT NULL`, `DEFAULT NOW()` |

**Entidad: `payment`**  

Propósito: Documentar el flujo financiero. Todo pago nace de una renta válida, registra monto exacto, instante de cobro y responsable, preservando trazabilidad contable.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `payment_id` | `IDENTIFIER` | Identificador único del cobro | Números enteros positivos generados por secuencia | `5090` | `PK`, `NOT NULL` |
| `customer_id` | `IDENTIFIER` | Cliente que realiza el pago | Referencia válida a `customer.customer_id` | `301` | `FK → customer`, `NOT NULL` |
| `staff_id` | `IDENTIFIER` | Empleado que procesó el cobro | Referencia válida a `staff.staff_id` | `14` | `FK → staff`, `NOT NULL` |
| `rental_id` | `IDENTIFIER` | Renta que originó el cobro | Referencia válida a `rental.rental_id` | `10402` | `FK → rental`, `NOT NULL` |
| `amount` | `DECIMAL(5,2)` | Monto monetario cobrado | Número decimal con dos cifras decimales, estrictamente positivo | `4.99` | `NOT NULL`, `CHECK (amount > 0)` |
| `payment_date` | `DATETIME` | Instante de procesamiento financiero | Fecha y hora válida (formato ISO 8601) | `2026-05-20 18:45:10` | `NOT NULL` |

#### Entidades Asociativas (Resolución de Relaciones Muchos-a-Muchos)

**Entidad: `film_actor`**  

Propósito: Resolver la relación entre actores y títulos. Garantiza que cada participación sea única y auditable.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `actor_id` | `IDENTIFIER` | Referencia al actor | Referencia válida a `actor.actor_id` | `42` | `FK → actor`, `NOT NULL` |
| `film_id` | `IDENTIFIER` | Referencia al título | Referencia válida a `film.film_id` | `105` | `FK → film`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-15 10:00:00` | `NOT NULL`, `DEFAULT NOW()` |
| **PK Compuesta** | | Combinación única | `(actor_id, film_id)` | `(42, 105)` | |

**Entidad: `film_category`**  

Propósito: Resolver la clasificación temática. Permite que un título pertenezca a múltiples categorías sin duplicar vínculos.

| Atributo | Tipo de Dato Abstracto | Significado | Dominio | Ejemplo | Restricciones |
|----------|------------------------|-------------|---------|---------|---------------|
| `film_id` | `IDENTIFIER` | Referencia al título | Referencia válida a `film.film_id` | `105` | `FK → film`, `NOT NULL` |
| `category_id` | `IDENTIFIER` | Referencia a la categoría | Referencia válida a `category.category_id` | `6` | `FK → category`, `NOT NULL` |
| `last_update` | `DATETIME` | Marca temporal de auditoría | Fecha y hora válida (formato ISO 8601) | `2026-05-15 10:05:00` | `NOT NULL`, `DEFAULT NOW()` |
| **PK Compuesta** | | Combinación única | `(film_id, category_id)` | `(105, 6)` | |

### 4.A.4. Matriz de Relaciones y Cardinalidades

El modelo no es un conjunto de tablas aisladas; es un ecosistema de dependencias. Cada línea de esta matriz indica cómo se comunican las entidades, cuántas instancias pueden vincularse y si esa vinculación es obligatoria o facultativa.

| Entidad Origen | Relación Semántica | Entidad Destino | Cardinalidad (Origen a Destino) | Participación | Mecanismo de Resolución |
|---------------|-------------------|-----------------|-----------------------------|---------------|------------------------|
| `actor` | participa en | `film` | (0,N) | Opcional | Entidad asociativa `film_actor` |
| `film` | clasificado como | `category` | (0,N) | Opcional | Entidad asociativa `film_category` |
| `film` | posee copias en | `inventory` | (1,N) | Total | `FK inventory.film_id → film.film_id` |
| `inventory` | ubicado en | `store` | (N,1) | Total | `FK inventory.store_id → store.store_id` |
| `store` | administrado por | `staff` | (1,1) | Total | `UNIQUE FK store.manager_staff_id → staff.staff_id` |
| `store` | emplea a | `staff` | (1,N) | Total | `FK staff.store_id → store.store_id` |
| `store` | registra a | `customer` | (1,N) | Total | `FK customer.store_id → store.store_id` |
| `customer` | realiza | `rental` | (0,N) | Opcional | `FK rental.customer_id → customer.customer_id` |
| `customer` | efectúa | `payment` | (0,N) | Opcional | `FK payment.customer_id → customer.customer_id` |
| `rental` | genera | `payment` | (1,N) | Total | `FK payment.rental_id → rental.rental_id` |
| `rental` | involucra | `inventory` | (N,1) | Total | `FK rental.inventory_id → inventory.inventory_id` |
| `rental` | procesada por | `staff` | (N,1) | Total | `FK rental.staff_id → staff.staff_id` |
| `payment` | procesada por | `staff` | (N,1) | Total | `FK payment.staff_id → staff.staff_id` |
| `staff` | reside en | `address` | (N,1) | Total | `FK staff.address_id → address.address_id` |
| `customer` | reside en | `address` | (N,1) | Total | `FK customer.address_id → address.address_id` |
| `store` | ubicada en | `address` | (N,1) | Total | `FK store.address_id → address.address_id` |
| `address` | pertenece a | `city` | (N,1) | Total | `FK address.city_id → city.city_id` |
| `city` | pertenece a | `country` | (N,1) | Total | `FK city.country_id → country.country_id` |
| `film` | expresado en | `language` | (N,1) | Total | `FK film.language_id → language.language_id` |
| `film` | originalmente en | `language` | (0,1) | Opcional | `FK film.original_language_id → language.language_id` |

**Nota de interpretación:** La participación total significa que no puede existir la entidad origen sin su destino. La participación opcional permite existencia independiente.

### 4.A.5. Restricciones de Integridad Lógica

Las reglas de negocio constituyen contratos matemáticos que el sistema debe respetar. Se agrupan en cuatro dimensiones operativas:

#### 4.A.5.1. Restricciones de Dominio

| ID | Atributo(s) | Regla Conceptual | Justificación Operativa |
|----|-------------|------------------|------------------------|
| RD-01 | `film.rating` | Pertenencia al conjunto `{'G','PG','PG-13','R','NC-17'}` | Política de contenido familiar; evita clasificaciones arbitrarias |
| RD-02 | `film.rental_rate`, `film.replacement_cost`, `payment.amount` | Mayor o igual a cero | Integridad financiera; un cobro negativo carece de sentido comercial |
| RD-03 | `film.rental_duration`, `film.length` | Estrictamente positivo | Duraciones nulas o negativas invalidan la lógica de renta y exhibición |
| RD-04 | `film.release_year` | Entre 1900 y año_actual + 5 | Rango histórico realista; bloquea datos erróneos de carga masiva |
| RD-05 | `customer.email`, `staff.email` | Formato RFC 5322 o nulo | Garantiza contactabilidad y evita cadenas no válidas en campos de contacto |
| RD-06 | `address.phone` | Formato numérico internacional o nulo | Permite validación automática y enrutamiento de notificaciones |

#### 4.A.5.2. Reglas de Integridad Referencial

| Clave Foránea | Entidad Referida | Política de Actualización | Política de Eliminación | Fundamento |
|--------------|-----------------|--------------------------|------------------------|------------|
| `film.language_id` | `language` | Propagar | Restringir | El idioma es esencial; no eliminar mientras existan títulos |
| `film.original_language_id` | `language` | Propagar | Establecer nulo | Opcional; anular referencia si se elimina el idioma |
| `store.manager_staff_id` | `staff` | Propagar | Restringir | Garantizar administración continua por sede |
| `store.address_id` | `address` | Propagar | Restringir | Ubicación física obligatoria para operaciones legales |
| `staff.address_id` | `address` | Propagar | Restringir | Domicilio crítico para contacto, nómina y seguridad |
| `staff.store_id` | `store` | Propagar | Restringir | Adscripción laboral obligatoria; no existen empleados flotantes |
| `customer.address_id` | `address` | Propagar | Restringir | Domicilio crítico para facturación y envío de comunicados |
| `customer.store_id` | `store` | Propagar | Restringir | Origen de registro obligatorio para segmentación comercial |
| `inventory.film_id` | `film` | Propagar | Restringir | Preservar inventario activo; no se eliminan copias con stock |
| `inventory.store_id` | `store` | Propagar | Restringir | Pertenencia física obligatoria; el inventario no flota |
| `rental.inventory_id` | `inventory` | Propagar | Restringir | Preservar historial transaccional; no se borra evidencia |
| `rental.customer_id` | `customer` | Propagar | Restringir | Trazabilidad financiera y operativa completa |
| `rental.staff_id` | `staff` | Propagar | Establecer nulo | Preservar transacción si el empleado deja la organización |
| `payment.rental_id` | `rental` | Propagar | Restringir | Vinculación financiera obligatoria; no existen pagos huérfanos |
| `payment.customer_id` | `customer` | Propagar | Restringir | Trazabilidad de pagos y conciliación contable |
| `payment.staff_id` | `staff` | Propagar | Establecer nulo | Preservar registro financiero ante rotación de personal |
| `address.city_id` | `city` | Propagar | Restringir | Jerarquía geográfica estricta; no existen direcciones sin ciudad |
| `city.country_id` | `country` | Propagar | Restringir | Jerarquía geográfica estricta; no existen ciudades sin país |
| `film_actor.actor_id` | `actor` | Propagar | Cascada | Eliminar asociaciones si se elimina al actor del registro |
| `film_actor.film_id` | `film` | Propagar | Cascada | Eliminar asociaciones si se elimina el título del catálogo |
| `film_category.film_id` | `film` | Propagar | Cascada | Eliminar clasificaciones si se elimina el título |
| `film_category.category_id` | `category` | Propagar | Restringir | Proteger catálogo temático; no se eliminan categorías activas |

**Nota de diseño:** La diferencia entre restringir y cascada es estratégica. Se utiliza restricción cuando la entidad maestra tiene dependencia operativa o financiera. Se utiliza cascada cuando la tabla asociativa solo existe por referencia a ambas entidades y su eliminación no compromete la integridad histórica.

### 4.A.6. Vistas Lógicas para Consulta y Reporte

Las vistas constituyen preguntas predefinidas que el sistema responde de manera automatizada. Permiten separar la complejidad del almacenamiento de la simplicidad del consumo de información.

| Vista | Propósito | Entidades Fuente | Proyección Lógica |
|-------|-----------|------------------|-------------------|
| `customer_list` | Directorio consolidado con ubicación geográfica completa | `customer`, `address`, `city`, `country`, `store` | `customer_id`, nombre completo, contacto, dirección estructurada, ciudad, país, tienda de origen, estado operativo |
| `staff_list` | Directorio operativo con adscripción y rol | `staff`, `address`, `city`, `country`, `store` | `staff_id`, nombre completo, contacto, dirección, ciudad, país, tienda, indicador de gestión, estado laboral |
| `film_list` | Catálogo enriquecido para exhibición y búsqueda | `film`, `language`, `category`, `film_category`, `actor`, `film_actor` | `film_id`, título, descripción, año, duración, tarifa, clasificación, idiomas, categorías agregadas, elenco agregado |
| `sales_by_film_category` | Agregación de ingresos por género temático | `payment`, `rental`, `inventory`, `film`, `film_category`, `category` | Categoría, ingresos totales, conteo de transacciones, período fiscal |
| `sales_by_store` | Comparativo de desempeño por sede | `payment`, `rental`, `customer`, `store`, `staff`, `address`, `city`, `country` | Sede (ciudad, país), administrador, ingresos totales, clientes activos, período |
| `rentals_by_category` | Métricas de popularidad y rotación | `rental`, `inventory`, `film`, `film_category`, `category` | Categoría, conteo de rentas, tasa de rotación implícita, período |

### 4.A.7. Validación de Normalización

La normalización constituye un mecanismo de ingeniería para prevenir inconsistencias. El modelo se valida contra las tres primeras formas normales:

| Forma Normal | Criterio de Cumplimiento | Evidencia en el Modelo |
|--------------|-------------------------|------------------------|
| **1FN** | Todos los atributos contienen valores atómicos; no existen grupos repetitivos. | Cada columna almacena un solo valor. Excepción documentada: `special_features` como colección por naturaleza enumerativa y baja granularidad de consulta individual. |
| **2FN** | Todos los atributos no clave dependen completamente de la clave primaria. | En `film_actor` y `film_category`, `last_update` depende de la combinación completa de claves, no de una parte aislada. |
| **3FN** | No existen dependencias transitivas entre atributos no clave. | La jerarquía geográfica se descompone en `address` a `city` a `country`. Ningún atributo no clave depende de otro no clave; todos dependen directamente de la clave primaria. |

**Perspectiva práctica:** En ocasiones se habla de desnormalización controlada. En este modelo, `film.fulltext` es un atributo derivado para búsquedas semánticas; no es fuente de verdad, se mantiene automáticamente. `customer.active` y `customer.activebool` coexisten por compatibilidad histórica y rendimiento de filtros frecuentes, pero la fuente de verdad transaccional reside en `rental` y `payment`.

### 4.A.7.1 Evidencia Empírica de Normalización

Para comprender por qué el diseño lógico adoptado en este estudio de caso no es un ejercicio teórico, sino una respuesta directa a problemas operativos reales, se analizan dos escenarios construidos a partir de los patrones de carga del script de referencia. Estos reportes contrastan cómo se comportaría la información en un modelo desnormalizado frente a la arquitectura relacional propuesta, demostrando empíricamente las anomalías que la normalización previene.

#### Caso 1: Jerarquía Geográfica y Direcciones Compartidas

En una cadena de renta, un mismo registro de dirección física puede ser compartido por múltiples actores del negocio: un cliente reside allí, un empleado vive en esa misma colonia y una tienda puede estar ubicada en el mismo distrito. Si el modelo intentara almacenar esta información en una tabla plana, la redundancia sería inevitable.

**Escenario Desnormalizado (Tabla Plana `entity_address_flat`)**

| `entity_type` | `entity_id` | `full_name` | `role` | `street_address` | `district` | `city_name` | `country_name` | `phone` |
|---------------|-------------|-------------|--------|------------------|------------|-------------|----------------|---------|
| customer | 104 | Maria Lopez | Rentadora | 1013 Tabuk Avenue | Southern Asia | Abha | Saudi Arabia | 9292223333 |
| staff | 12 | Ahmed Al-Faisal | Cajero | 1013 Tabuk Avenue | Southern Asia | Abha | Saudi Arabia | 9292223333 |
| store | 2 | Pagila Store 02 | Sede Operativa | 1013 Tabuk Avenue | Southern Asia | Abha | Saudi Arabia | 9292223333 |

**Anomalías Identificadas:**

- **Redundancia Masiva:** `city_name`, `country_name`, `district` y `phone` se repiten idénticamente. En un volumen real, esto dispara el almacenamiento y satura las consultas con tráfico innecesario.
- **Anomalía de Actualización:** Si `Abha` cambia su denominación oficial a `Al-Abha`, se deben modificar N filas. Si una omisión ocurre, los reportes geográficos muestran inconsistencias críticas.
- **Anomalía de Inserción o Eliminación:** No se puede registrar una ciudad nueva sin asociarla obligatoriamente a un cliente o tienda, y eliminar al último cliente de esa dirección borraría accidentalmente la ubicación de la tienda y del empleado.

**Solución:** La jerarquía `address` a `city` a `country` desacopla la ubicación de los actores. Un cambio en `city.city` se refleja transversalmente mediante una única actualización y operación de unión. Se garantiza 1FN, 2FN y 3FN de manera natural, preservando la integridad sin duplicar datos.

#### Caso 2: Catálogo de Películas, Actores, Categorías e Idiomas

Una película puede tener múltiples actores, pertenecer a varias categorías y contar con metadatos comerciales y técnicos. Aplanar esta estructura genera violaciones formales graves que colapsan la mantenibilidad del sistema.

**Escenario Desnormalizado (Tabla Plana `film_catalog_flat`)**

| `film_id` | `title` | `release_year` | `length` | `rental_rate` | `language` | `original_language` | `category` | `actor_name` | `special_features_list` |
|-----------|---------|----------------|----------|---------------|------------|---------------------|------------|--------------|-------------------------|
| 1 | ACADEMY DINOSAUR | 2006 | 86 | 0.99 | English | English | Documentary | PENELOPE GUINESS | Deleted Scenes, Trailers |
| 1 | ACADEMY DINOSAUR | 2006 | 86 | 0.99 | English | English | Documentary | CHRISTIAN GABLE | Deleted Scenes, Trailers |
| 1 | ACADEMY DINOSAUR | 2006 | 86 | 0.99 | English | English | Horror | PENELOPE GUINESS | Deleted Scenes, Trailers |
| 1 | ACADEMY DINOSAUR | 2006 | 86 | 0.99 | English | English | Horror | CHRISTIAN GABLE | Deleted Scenes, Trailers |

**Anomalías Identificadas:**

- **Violación de 1FN:** `special_features_list` agrupa múltiples valores en una celda, imposibilitando filtros eficientes sin operaciones de procesamiento costosas.
- **Violación de 2FN:** `actor_name` y `category` no dependen de la clave primaria compuesta implícita, sino de entidades externas. Actualizar `rental_rate` obliga a modificar 4 filas con alto riesgo de inconsistencia transaccional.
- **Violación de 3FN:** `language` depende de `language_id`, no de la fila combinada. Cambios en catálogos maestros requieren actualizaciones masivas proporcionalmente al producto cartesiano de actores por categorías.

**Solución:** El esquema descompone el dominio en `film`, `language`, `actor`, `category` y tablas asociativas (`film_actor`, `film_category`). Cada tabla almacena solo atributos que dependen funcionalmente de su clave primaria. Se garantiza atomicidad, independencia de claves y trazabilidad semántica, permitiendo que el catálogo evolucione sin reestructuraciones.

**Reflexión Pedagógica:** Estos reportes no son ejercicios hipotéticos; reflejan problemas operativos reales que colapsaron sistemas de gestión en la era analógica. La normalización no es un dogma académico, sino un mecanismo de ingeniería que transforma la complejidad del negocio en estructuras predecibles, auditables y escalables. Cada decisión de separación en el diseño lógico responde directamente a una anomalía documentada aquí, garantizando que el sistema crezca sin romperse y mantenga la coherencia semántica a lo largo de su ciclo de vida.

### 4.A.8. Matriz de Trazabilidad Bidireccional

Para que el estudio de caso no sea un ejercicio aislado, cada estructura lógica debe poder rastrearse hasta una necesidad documentada. Esta matriz cierra el ciclo: de requisito a modelo, y de modelo a implementación.

| Requisito (Parte 3) | Entidad o Relación Lógica Asociada | Tipo de Requisito | Estado de Implementación Lógica |
|---------------------|----------------------------------|-------------------|--------------------------------|
| RF-01: Catálogo Maestro | `film`, `language` | Funcional | Completo (atributos descriptivos, FK a idioma, restricciones de dominio) |
| RF-02: Asociación M:N | `film_actor`, `film_category` | Funcional o Datos | Completo (PK compuestas, unicidad garantizada, auditoría temporal) |
| RF-03: Inventario por Sede | `inventory`, `rental` | Funcional | Completo (FK a `film` y `store`, restricción de exclusividad temporal) |
| RF-04: Renta o Devolución | `rental` | Funcional | Completo (marca temporal opcional `return_date`, preservación histórica) |
| RF-05: Pagos Validados | `payment` | Funcional o Datos | Completo (FK obligatoria a `rental`, validación de monto positivo) |
| RF-06: Normalización Geográfica | `address` a `city` a `country` | Funcional | Completo (jerarquía N:1, propagación de cambios, restricciones de eliminación) |
| RF-07: Reportes Analíticos | Vistas lógicas `sales_by_*` | Funcional o No Funcional | Completo (agregaciones definidas conceptualmente, independientes de entorno) |
| RNF-01: Disponibilidad o Atomicidad | Transacciones `rental`, `payment` | No Funcional | Soportado por diseño de integridad referencial estricta |
| RNF-02: Desempeño | Índices lógicos implícitos en PK o FK | No Funcional | Estructura optimizada para uniones frecuentes y validación de unicidad |
| RNF-03: Control de Accesos | `staff.username`, `password`, `active` | No Funcional | Modelado de credenciales y estado lógico; auditoría mediante `last_update` |
| RNF-04: Escalabilidad | Identificadores sustitutos, 3FN | No Funcional | Esquema extensible por adición de entidades sin reestructuración |
| RNF-05: Soberanía o Privacidad | `customer`, `staff`, `address` | No Funcional | Datos personales normalizados, trazabilidad temporal, soporte para anonimización lógica |

### 4.A.9. Control de Versiones del Anexo

| Versión | Fecha | Autor | Cambios Principales |
|---------|-------|-------|---------------------|
| 1.0 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Especificación completa del diseño lógico; consolidación de entidades, relaciones, restricciones y vistas; validación de normalización y trazabilidad. |
| 1.1 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Reestructuración de tablas de entidades: separación de propósito, columnas `Significado` y `Ejemplo`, renombrado de tipos a abstractos, integración de notas pedagógicas. |
| 1.2 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Adición de sección 4.A.7.1 con evidencia empírica de normalización usando patrones reales de carga; eliminación de referencias a tecnologías específicas; explicitación del fundamento relacional. |
| 1.3 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Reemplazo de columna `Ejemplo` por `Dominio` en todas las entidades para consignar formalmente los valores posibles; eliminación de símbolos decorativos; formalización de la notación académica. |
| 1.4 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Restauración de columna `Ejemplo` junto a `Dominio` para ilustración concreta; adición de etiqueta `Entidad:` para mayor claridad estructural del reporte. |

---

**Declaración de Propósito Académico:**  
Este anexo ha sido elaborado exclusivamente con fines pedagógicos dentro del estudio de caso de la UEA Bases de Datos (2151106) de la Universidad Autónoma Metropolitana, Unidad Iztapalapa. Su contenido constituye la especificación formal e independiente de tecnología que sirve de base para el diseño físico del esquema de referencia. Se ha adoptado explícitamente el modelo relacional como paradigma de diseño, pero se mantiene deliberadamente agnóstico respecto a cualquier motor de almacenamiento, lenguaje de consulta o plataforma de implementación. No prescribe tecnologías ni constituye documento contractual.

**Elaborado para distribución educativa**  
**Universidad Autónoma Metropolitana - Unidad Iztapalapa**  
*Licenciatura en Computación | UEA Bases de Datos (2151106)*  
*Documento de distribución educativa bajo licencia CC BY-NC-SA 4.0. Se autoriza la reproducción parcial con citación institucional completa.*
