# Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas  
## Parte 4: Diseño Lógico de la Base de Datos  

**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Licenciatura:** Computación  
**UEAs:** Bases de Datos (2151106), Análisis y Diseño de Sistemas, Ingeniería de Software  
**Autor:** Dr. Jesús Zavala Ruiz  
**Última Actualización:** 22 de mayo de 2026  

---

### 4.1. Introducción

Imaginemos por un momento que hemos comprendido profundamente cómo opera el negocio de renta de películas: conocemos sus actores, sus procesos, sus reglas y sus necesidades de información. Sabemos qué debe hacer el sistema, para quién y bajo qué condiciones. Pero aún no hemos dado el paso crucial: traducir ese entendimiento conceptual en una estructura de datos formal, rigurosa y verificable. Ese es el propósito del diseño lógico.

El diseño lógico de una base de datos es el arte y la ciencia de transformar los requisitos del negocio en un modelo de datos independiente de tecnología. No se trata aún de decidir si usaremos PostgreSQL, MySQL o cualquier otro motor; se trata de definir *qué* datos existen, *cómo* se relacionan entre sí y *qué reglas* deben respetar para garantizar integridad, consistencia y significado. Es, en esencia, la cristalización estructural del dominio.

Este proceso rara vez es lineal. La experiencia en ingeniería de datos nos enseña que el modelado lógico es inherentemente iterativo: se requieren entre tres y cinco ciclos de refinamiento para estabilizar un diseño robusto. En el primer ciclo, identificamos entidades y relaciones gruesas; en el segundo, refinamos atributos y cardinalidades; en el tercero, validamos normalización y restricciones; en ciclos posteriores, optimizamos para consultas frecuentes y anticipamos evolución. Cada iteración acerca el modelo a una representación fiel del negocio, libre de ambigüedades y preparada para su materialización técnica.

Uno de los desafíos recurrentes en esta fase es la diversidad de notaciones disponibles para representar modelos lógicos. El diagrama Entidad-Relación original propuesto por Peter Chen en 1976 ofrece una sintaxis gráfica elegante para conceptos fundamentales; la notación Crow's Foot (pata de gallo) se ha popularizado en herramientas CASE por su claridad visual para cardinalidades; y los diagramas de clases de UML permiten integrar el modelado de datos con el diseño orientado a objetos. En este estudio de caso, adoptamos una convención textual basada en nombres de entidades y atributos en inglés con notación `snake_case`, alineada con el esquema de referencia `pagila`. Esta decisión no es arbitraria: responde a criterios de interoperabilidad, consistencia con la literatura técnica internacional y trazabilidad directa con implementaciones reales. Los anexos de este documento ejemplifican las tres notaciones mencionadas para fines comparativos y pedagógicos.

Lo que sigue es la especificación completa del diseño lógico del sistema de renta de películas. Cada entidad, atributo, relación y restricción ha sido derivado sistemáticamente de los requisitos documentados en la Parte 3, garantizando que ninguna decisión de modelado quede desconectada de una necesidad operativa real. Al finalizar esta lectura, el estudiante habrá comprendido no solo *qué* estructura de datos soporta el negocio, sino *por qué* cada elemento existe y *cómo* se articula con el resto para formar un todo coherente.

### 4.2. Convenciones de Modelado

Antes de describir la estructura lógica, es fundamental establecer las convenciones que guiarán su representación. Estas decisiones no son meramente estéticas; condicionan la legibilidad, la mantenibilidad y la interoperabilidad del modelo a lo largo de su ciclo de vida.

**Nombres de entidades y atributos en inglés:**  
Aunque el negocio se describe en español y los documentos conceptuales se redactan en nuestro idioma, los nombres de entidades y atributos se especifican en inglés (`film`, `rental_date`, `manager_staff_id`). Esta práctica responde a tres razones fundamentales: primero, alinea el modelo con la literatura técnica internacional y los estándares de la industria, donde el inglés funciona como lingua franca; segundo, facilita la trazabilidad directa con el esquema de referencia `pagila`, cuyos objetos ya utilizan esta convención; tercero, anticipa la implementación en sistemas gestores que, aunque soporten identificadores en Unicode, operan con mayor fluidez y compatibilidad cuando los nombres siguen patrones anglosajones.

**Notación `snake_case`:**  
Los nombres compuestos se escriben en minúsculas con separación por guion bajo (`rental_date`, `manager_staff_id`, `special_features`). Esta convención, ampliamente adoptada en PostgreSQL y en la comunidad de código abierto, mejora la legibilidad al separar visualmente los componentes semánticos de un identificador, evita conflictos con palabras reservadas que suelen usar mayúsculas y garantiza consistencia en entornos donde la sensibilidad a mayúsculas/minúsculas puede variar.

**Identificadores con sufijo `_id`:**  
Toda entidad maestra o transaccional posee un atributo identificador con el sufijo `_id` (`film_id`, `customer_id`, `rental_id`). Esta práctica establece un patrón reconocible que facilita la navegación del modelo, la construcción de claves foráneas y la auditoría de integridad referencial.

**Tipos de datos genéricos:**  
En esta fase lógica, los tipos de datos se especifican de manera abstracta (`IDENTIFIER`, `STRING(n)`, `DECIMAL(p,s)`, `DATETIME`), sin vincularse a implementaciones específicas de ningún SGBD. Esta independencia conceptual es esencial para garantizar que el diseño pueda materializarse en múltiples plataformas tecnológicas sin pérdida semántica.

Con estas convenciones establecidas, procedemos a describir la estructura lógica del sistema.

### 4.3. Especificación de Entidades y Atributos

El modelo lógico identifica trece entidades nucleares que representan los conceptos fundamentales del dominio, más dos entidades asociativas que resuelven relaciones muchos-a-muchos. Cada entidad se describe mediante su propósito, sus atributos con tipos genéricos y restricciones, y las reglas de negocio que la condicionan.

#### 4.3.1. Entidades Maestras del Catálogo

La entidad `actor` representa a las personas que participan en producciones audiovisuales. Posee un identificador invariante `actor_id`, nombres propios `first_name` y `last_name` obligatorios, y una marca temporal `last_update` que documenta su última modificación. La regla de negocio subyacente exige que todo actor registrado posea al menos un nombre y un apellido, garantizando identificación mínima para fines operativos.

La entidad `film` constituye el núcleo del catálogo: representa un título audiovisual disponible para renta. Además de su identificador `film_id` y título obligatorio `title`, almacena descripción opcional `description`, año de estreno `release_year`, referencias a idiomas (`language_id` obligatorio, `original_language_id` opcional), parámetros comerciales (`rental_duration`, `rental_rate`, `replacement_cost`), duración `length`, clasificación etaria `rating` restringida a un catálogo predefinido, características especiales `special_features` como colección de valores, y un atributo derivado `fulltext` para búsqueda semántica. La marca temporal `last_update` garantiza trazabilidad de cambios.

Las entidades `category` e `language` funcionan como catálogos maestros: la primera clasifica títulos por afinidad temática mediante un nombre único `name`; la segunda enumera idiomas soportados, también con denominación única. Ambas comparten la estructura mínima de identificador, nombre y marca de auditoría.

#### 4.3.2. Entidades de Infraestructura Geográfica y Organizacional

La jerarquía geográfica se modela mediante tres entidades encadenadas: `country`, `city` y `address`. Cada país posee un nombre único; cada ciudad refiere obligatoriamente a un país; cada dirección refiere a una ciudad y almacena componentes estructurados (`address`, `address2`, `district`, `postal_code`, `phone`). Esta normalización evita redundancia: un cambio en el nombre de una ciudad se refleja transversalmente en todas las direcciones que la referencian.

La entidad `store` representa un establecimiento físico de renta. Vincula obligatoriamente un administrador único (`manager_staff_id` con restricción de unicidad) y una dirección válida (`address_id`). La entidad `staff` modela al personal operativo: además de identificación personal y domicilio, registra credenciales de acceso (`username` único, `password` encriptado), tienda de adscripción y estado laboral `active`.

#### 4.3.3. Entidades Transaccionales y de Relación con el Cliente

La entidad `customer` representa a los usuarios finales del servicio. Se asocia obligatoriamente a una tienda de origen (`store_id`) y a una dirección (`address_id`), almacena datos de contacto y registra su estado operativo mediante `activebool`. La fecha de alta `create_date` y la marca de auditoría `last_update` completan su perfil.

La entidad `inventory` materializa el concepto de copia física distinguible: cada ejemplar de un título en una tienda posee identificador propio `inventory_id`, refiere al título (`film_id`) y a la sede (`store_id`), y documenta su última modificación. Esta entidad es crucial para controlar disponibilidad en tiempo real.

La entidad `rental` captura la transacción nuclear del negocio: registra fecha y hora de inicio `rental_date`, ejemplar rentado `inventory_id`, cliente `customer_id`, empleado procesador `staff_id` y, opcionalmente, fecha de devolución `return_date`. La regla de exclusividad temporal impide que un mismo ejemplar sea asignado a múltiples rentas en el mismo instante.

Finalmente, la entidad `payment` documenta los cobros asociados a rentas: vincula obligatoriamente cliente, empleado y renta de origen, registra monto positivo `amount` y fecha exacta `payment_date`. La integridad referencial garantiza que ningún pago exista sin una renta válida que lo justifique.

#### 4.3.4. Entidades Asociativas para Relaciones Muchos-a-Muchos

Dos relaciones del dominio requieren resolución mediante entidades intermedias. La asociación entre `actor` y `film` es muchos-a-muchos: un actor participa en múltiples títulos y un título involucra a múltiples actores. La entidad `film_actor` resuelve esta relación mediante una clave primaria compuesta (`actor_id`, `film_id`) que garantiza unicidad en la asociación, más una marca de auditoría `last_update`.

Análogamente, la clasificación temática de títulos se modela mediante `film_category`, que vincula `film_id` y `category_id` en una clave compuesta única. Esta estructura permite que un título pertenezca a múltiples categorías sin duplicidad de vínculos.

### 4.4. Especificación de Relaciones y Restricciones de Integridad

El modelo lógico no solo define entidades aisladas, sino que articula un entramado de relaciones semánticas condicionadas por reglas de integridad. Cada relación se especifica mediante cardinalidad, participación y mecanismo de resolución.

La relación entre `actor` y `film` es opcional en ambos extremos y de cardinalidad muchos-a-muchos, resuelta mediante `film_actor`. La eliminación de un actor o título propaga en cascada a las asociaciones, preservando consistencia.

La relación entre `film` y `category` sigue el mismo patrón muchos-a-muchos, resuelta por `film_category`. Sin embargo, la eliminación de una categoría se restringe si existen títulos clasificados, protegiendo la integridad del catálogo temático.

La jerarquía geográfica `address` → `city` → `country` establece relaciones muchos-a-uno con participación total hacia arriba: toda dirección refiere a una ciudad válida, toda ciudad a un país válido. La eliminación de niveles superiores se restringe mientras existan dependencias activas.

La relación entre `store` y `staff` presenta dos facetas: una asociación uno-a-uno para el rol de administrador (`manager_staff_id` con unicidad) y una relación uno-a-muchos para el empleo general. Esta dualidad refleja la distinción operativa entre responsabilidad administrativa y adscripción laboral.

Las transacciones de renta y pago establecen cadenas de dependencia estricta: `rental` refiere obligatoriamente a `inventory`, `customer` y `staff`; `payment` refiere obligatoriamente a `rental`, `customer` y `staff`. La eliminación de registros maestros se restringe si existen transacciones activas, garantizando preservación del historial financiero.

Además de las restricciones referenciales, el modelo incorpora validaciones de dominio: la clasificación etaria `film.rating` se restringe al conjunto {'G','PG','PG-13','R','NC-17'}; los valores monetarios (`rental_rate`, `replacement_cost`, `amount`) deben ser no negativos; las duraciones (`rental_duration`, `length`) deben ser positivas. Estas reglas se expresan de manera abstracta, independientes de sintaxis de lenguaje de definición de datos.

### 4.5. Principios de Normalización y Decisiones de Diseño

El diseño lógico aplica rigurosamente los principios de normalización relacional para eliminar redundancias y anomalías. La Primera Forma Normal se cumple al almacenar valores atómicos en todos los atributos, con la excepción controlada de `film.special_features`, modelado como colección de valores discretos por su naturaleza enumerativa y bajo volumen de consulta individual.

La Segunda Forma Normal se verifica en entidades con clave compuesta: en `film_actor` y `film_category`, el atributo `last_update` depende de la combinación completa de claves, no de una parte aislada. La Tercera Forma Normal se garantiza al eliminar dependencias transitivas: la jerarquía geográfica se descompone en entidades independientes (`address`, `city`, `country`) para que atributos no clave no dependan de otros atributos no clave.

Ciertas decisiones de diseño merecen justificación explícita. El uso de identificadores sustitutos (`film_id`, `actor_id`, etc.) en lugar de claves naturales garantiza estabilidad referencial ante cambios en datos descriptivos. La estrategia de auditoría temporal mediante `last_update` en todas las entidades maestras y transaccionales facilita sincronización y trazabilidad, aunque no identifica al autor del cambio. El manejo de estados activos/inactivos mediante banderas lógicas (`active`, `activebool`) permite desactivación sin eliminación física, preservando historiales transaccionales.

Finalmente, el modelo anticipa necesidades de consulta mediante vistas lógicas predefinidas: `customer_list` y `staff_list` consolidan información geográfica para directorios; `film_list` enriquece el catálogo con actores y categorías; `sales_by_film_category` y `sales_by_store` agregan ingresos para análisis comercial. Estas vistas se especifican conceptualmente, independientes de sintaxis de lenguaje de consulta.

### 4.6. Trazabilidad: De los Requisitos al Modelo Lógico

La coherencia del diseño lógico se valida mediante trazabilidad explícita hacia los requisitos documentados en la Parte 3. Cada entidad, atributo y restricción puede rastrearse hasta una necesidad operativa del negocio:

- La entidad `film` y sus atributos descriptivos derivan del requisito RF-01 (Gestión del Catálogo Maestro de Títulos).
- Las entidades asociativas `film_actor` y `film_category` materializan el requisito RF-02 (Asociación de Títulos con Participantes y Categorías).
- La entidad `inventory` y la restricción de unicidad temporal en `rental` responden al requisito RF-03 (Control de Inventario Físico por Establecimiento).
- La cadena transaccional `rental` → `payment` y la preservación de historial satisfacen los requisitos RF-04 y RF-05.
- La jerarquía geográfica normalizada (`address` → `city` → `country`) implementa el requisito RF-06 (Normalización de Información Geográfica).
- Las vistas lógicas para reportes operativos y analíticos habilitan el requisito RF-07.

Esta trazabilidad bidireccional garantiza que el modelo lógico no sea un ejercicio abstracto, sino la cristalización verificable de necesidades reales del negocio.

### 4.7. Consideraciones de Evolución y Extensibilidad

El diseño lógico anticipa escenarios de evolución mediante principios de extensibilidad controlada. La estructura normalizada permite incorporar nuevas sedes, expandir el catálogo y acumular historial sin reestructuración de fondo. Los identificadores sustitutos facilitan migraciones futuras y integración con sistemas externos.

Puntos de extensión identificados incluyen: una entidad `streaming_access` para rentas digitales; una tabla `reservation` para gestión de reservas; una estructura `loyalty_point` para programas de fidelización; y una entidad `review` para reseñas de clientes. Cada extensión seguiría el principio de aditividad: agregar entidades o atributos sin modificar estructuras existentes, preservando compatibilidad hacia atrás.

### 4.8. Conclusiones

El diseño lógico presentado constituye el puente formal entre la conceptualización del negocio y la materialización técnica. Al especificar entidades, atributos, relaciones y restricciones de manera independiente de tecnología, hemos establecido una base robusta, verificable y trazable para la fase de diseño físico. Cada decisión de modelado responde a una necesidad operativa documentada; cada estructura lógica anticipa requisitos de integridad, desempeño y evolución.

En las fases siguientes del estudio de caso, este modelo se materializará en un esquema PostgreSQL, validando que la implementación técnica corresponda punto por punto con la especificación lógica. Pero antes de dar ese paso, conviene reflexionar sobre las distintas notaciones disponibles para representar modelos lógicos. Los anexos que siguen ejemplifican tres enfoques complementarios—Chen, Crow's Foot y UML—para enriquecer la comprensión pedagógica del modelado de datos.

---

### Anexos: Notaciones para Representación de Modelos Lógicos

#### Anexo 4.A: Documento de Diseño Lógico Detallado Completo

[link]


#### Anexo 4.B: Diagrama Entidad-Relación en la Notación de Chen (1976)

La notación original propuesta por Peter Chen representa entidades mediante rectángulos, atributos mediante óvalos vinculados a sus entidades, y relaciones mediante rombos conectados a las entidades participantes. Las cardinalidades se indican mediante etiquetas (1, N, M) en los enlaces. Esta notación es pedagógicamente valiosa por su claridad conceptual, aunque menos práctica para modelos complejos debido a la proliferación de óvalos en entidades con muchos atributos.

[Desarrollar completa con código Mermaid]

#### Anexo 4.C: Diagrama Entidad-Relación en la Notación Crow's Foot (Pata de Gallo)

La notación Crow's Foot, ampliamente adoptada en herramientas CASE modernas, representa entidades mediante rectángulos que listan atributos internamente, y relaciones mediante líneas con símbolos gráficos en los extremos que indican cardinalidad y participación: una barra vertical para "uno", un círculo para "cero o uno", una pata de gallo (tres líneas divergentes) para "muchos". Esta notación es visualmente compacta y eficiente para modelos de mediana a gran complejidad.

[Desarrollar completa con código Mermaid a partir de la imagen disponible en https://github.com/devrimgunduz/pagila/blob/master/pagila-schema-diagram.png ]

[Incluir ambas imágenes]

#### Anexo 4.D: Diagrama de Clases UML para Modelado de Datos

El lenguaje UML, originalmente diseñado para ingeniería de software orientada a objetos, puede adaptarse para modelado de datos mediante clases que representan entidades, atributos listados en compartimentos internos, y asociaciones con multiplicidad en los extremos. Esta notación facilita la integración entre modelado de datos y diseño de aplicación, aunque requiere disciplina para evitar mezclar preocupaciones de persistencia y comportamiento.

[Desarrollar el diagrama de clases completo con código mermaid]

---

### Control de Versiones

| Versión | Fecha | Autor | Cambios Principales |
|---------|-------|-------|---------------------|
| 1.0 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Especificación inicial del diseño lógico derivada de requisitos (Parte 3); justificación de convenciones de nomenclatura; inclusión de anexos comparativos de notaciones. |
| 1.1 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Revisión para mejorar fluidez narrativa, reforzar trazabilidad hacia requisitos y consolidar ejemplos de notaciones en anexos. |

---

> **Declaración de Propósito Académico:**  
> Este documento ha sido elaborado exclusivamente con fines pedagógicos dentro del estudio de caso de la UEA Bases de Datos (2151106) de la Universidad Autónoma Metropolitana, Unidad Iztapalapa. Su contenido especifica hipotéticamente el diseño lógico independiente de tecnología que podría haber dado origen al esquema `pagila`, integrando principios de modelado relacional y normalización. No constituye un documento contractual ni prescribe tecnologías de implementación específicas.

---

**Elaborado para distribución educativa**  
**Universidad Autónoma Metropolitana - Unidad Iztapalapa**  
*Licenciatura en Computación | UEA Bases de Datos (2151106)*  
*Documento de distribución educativa bajo licencia CC BY-NC-SA 4.0. Se autoriza la reproducción parcial con citación institucional completa.*
