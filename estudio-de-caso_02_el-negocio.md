# Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas  
## Parte 2: El Negocio de Renta de Películas  

**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Licenciatura:** Computación  
**UEAs:** Bases de Datos (2151106), Análisis y Diseño de Sistemas, Ingeniería de Software  
**Autor:** Dr. Jesús Zavala Ruiz  
**Última Actualización:** 22 de mayo de 2026  

---

### 2.1. Propósito y Alcance Conceptual

Imagina por un momento que es 1995. Es viernes por la tarde y decides rentar una película. Entras a una tienda iluminada, recorres estanterías repletas de cajas de VHS organizadas por género, conversas con un empleado que conoce tus preferencias y sales con tu selección bajo el brazo. Este ritual, aparentemente simple, escondía una complejidad operativa extraordinaria: miles de títulos, cientos de copias físicas, decenas de empleados y miles de clientes cuyas interacciones debían coordinarse sin errores, sin duplicidades y sin perder el rastro de ninguna transacción.

Este documento nace de esa imagen. Su propósito es reconstruir, con claridad y trazabilidad, cómo operaba una cadena de establecimientos dedicados a la renta de contenido audiovisual en formato físico, inspirada en el modelo histórico de Blockbuster. Pero no se trata de un ejercicio nostálgico. Se trata de entender las dinámicas operativas, las reglas de negocio y las necesidades de información que justifican el desarrollo de un sistema automatizado de gestión. Cada decisión que tomaremos en las fases de diseño lógico y físico deberá poder rastrearse hasta una necesidad real de este negocio ficticio.

Este artefacto marca el inicio del ciclo de desarrollo de sistemas dentro de nuestra metodología. A partir de esta descripción del negocio, derivaremos progresivamente los requisitos del sistema, el diseño lógico y finalmente el diseño físico, cerrando el ciclo al validar que nuestra reconstrucción conceptual corresponde con el esquema de base de datos original.

En las páginas que siguen encontrarás una descripción del modelo de negocio, su propuesta de valor y sus fuentes de ingreso; la identificación de los actores organizacionales, los procesos clave y los activos que importan; las restricciones operativas y el marco de cumplimiento que condicionan el diseño; y, finalmente, las necesidades de información que motivan la automatización. Quedan fuera, deliberadamente, las decisiones sobre tecnologías específicas, los detalles de interfaces de usuario y las consideraciones legales particulares de alguna jurisdicción. Nuestro foco es el *qué* y el *por qué*, no el *cómo* técnico.

### 2.2. El Modelo de Negocio: Inspiración en una Era Analógica

Para dar vida a este caso de estudio, nos inspiramos en el modelo operativo de Blockbuster, la cadena que dominó la renta de contenido audiovisual en formato físico durante las décadas de 1980 y 1990. Sin embargo, no buscamos replicar históricamente su operación, sino extraer los elementos estructurales que resultan pedagógicamente valiosos para el aprendizaje de análisis y diseño de sistemas.

El negocio que conceptualizamos operaba con un catálogo ampliado: cada tienda exhibía más de mil títulos, muy por encima de las quinientas referencias típicas de un videoclub independiente. Esta amplitud no era un lujo, sino una ventaja competitiva que exigía un sistema capaz de almacenar, clasificar y recuperar eficientemente metadatos descriptivos de un volumen considerable de contenidos. Para lograrlo, la cadena establecía acuerdos directos con productoras, obteniendo películas a menor costo mediante la distribución de ingresos: sesenta por ciento para la cadena, cuarenta por ciento para el productor. Este modelo de adquisición por lotes generaba la necesidad de registrar términos comerciales, calcular participaciones económicas y mantener trazabilidad entre títulos y acuerdos de adquisición.

La experiencia del cliente se diseñaba alrededor de la exhibición en estantería: todo el catálogo estaba visible y accesible, a diferencia del modelo tradicional de mostrador cerrado. Esta decisión operativa, aparentemente simple, exigía un mecanismo de control de disponibilidad en tiempo real que evitara asignar el mismo ejemplar a múltiples clientes simultáneamente. Además, el negocio enfocaba su estrategia en contenido de nicho: aproximadamente el setenta por ciento de los alquileres correspondía a producciones de serie B o directas para video, no solo a estrenos de grandes estudios. Esta realidad demandaba capacidades de clasificación flexible, búsqueda por múltiples criterios y soporte para descripciones temáticas complejas.

La economía del modelo se sostenía en una premisa sencilla: el alto costo de adquisición de cintas originales hacía que el alquiler fuera más rentable para el cliente final que la compra. Esta dinámica justificaba preservar históricamente las transacciones, analizar la rotación de inventario y generar métricas de popularidad para orientar decisiones de adquisición. La gestión se basaba en datos: se registraban estadísticas de los títulos más rentados para orientar compras, controlar inventario y gestionar devoluciones, lo que generaba la necesidad de reportes agregados por período, categoría y ubicación, así como trazabilidad temporal de los cambios en los registros.

Finalmente, el negocio gestionaba el ciclo de vida completo de sus activos físicos. Las copias retiradas de circulación o sobrantes se ponían a la venta como fuente de ingreso secundaria, lo que requería gestionar estados transicionales que reflejaran su disponibilidad comercial. Y, por política empresarial explícita, ningún establecimiento podía comercializar contenido para adultos, lo que imponía restricciones de dominio sobre la clasificación de contenidos, garantizando que ningún registro violara los criterios establecidos.

Estos elementos no son meras curiosidades históricas; cada uno plantea desafíos concretos de modelado de información que el sistema deberá resolver. A continuación, exploramos cómo se traducen en propuesta de valor y fuentes de ingreso.

### 2.3. Valor y Fuentes de Ingresos: Lo que el Cliente Recibe y lo que el Negocio Captura

#### 2.3.1. Valor Central: Más que Rentar Películas

En esencia, el negocio busca ofrecer acceso inmediato, confiable y curado a un catálogo extenso de contenido audiovisual en formato físico. Esto se logra mediante una red de establecimientos locales con inventario gestionado en tiempo real, atención presencial personalizada y políticas de servicio transparentes. No se trata solo de rentar películas; se trata de generar confianza, conveniencia y experiencia en cada interacción con el cliente.

#### 2.3.2. Dimensiones de Valor: Lo que el Cliente Espera y lo que el Sistema debe Habilitar

La amplitud de catálogo significa para el cliente poder encontrar fácilmente entre más de mil títulos por tienda, clasificados por género, idioma, clasificación etaria y año. Para habilitar esta experiencia, el sistema debe almacenar descripciones estructuradas de cada título, asociar múltiples clasificaciones temáticas, registrar participantes del elenco y permitir búsquedas combinadas.

La disponibilidad garantizada permite al cliente saber, antes de llegar al mostrador, si el ejemplar que busca está disponible. El sistema responde identificando de manera única cada ejemplar físico, registrando su ubicación por establecimiento y verificando su estado de ocupación al procesar una solicitud.

La experiencia presencial se traduce en recibir asesoría personalizada y completar el proceso de renta o devolución de manera ágil en el mostrador. El sistema la soporta registrando información de contacto de clientes, gestionando credenciales de acceso para el personal y vinculando cada transacción con el empleado responsable.

La transparencia comercial permite al cliente conocer claramente las tarifas, los cargos por reposición y acceder a su historial de pagos sin ambigüedades. El sistema la garantiza vinculando explícitamente cada cobro con la transacción que lo origina, registrando montos con precisión monetaria y preservando el historial financiero.

Finalmente, la curaduría temática ayuda al cliente a descubrir contenidos afines a sus intereses mediante clasificaciones múltiples y búsquedas por elenco o características. El sistema la habilita permitiendo asociar un título con múltiples categorías y participantes, almacenando características adicionales como colecciones de valores y habilitando búsquedas textuales semánticas.

#### 2.3.3. Fuentes de Ingreso: Cómo se Genera Valor Económico

El negocio captura valor a través de múltiples mecanismos. La renta por período aplica un cargo fijo por cada ciclo de préstamo autorizado, por ejemplo tres días. Para soportarlo, el sistema debe capturar fecha y hora de inicio, duración contractual, tarifa aplicable, identificación del ejemplar, del cliente y del empleado que procesó.

El cargo por reposición se activa cuando un ejemplar se pierde o se daña sin posibilidad de recuperación. El sistema debe registrar el valor de reposición asociado al título, la fecha del cobro y el vínculo con la transacción de renta que no registró devolución.

La venta de excedentes comercializa ejemplares retirados de circulación o con inventario obsoleto como ingreso secundario. El sistema debe gestionar el estado comercial del ejemplar, la fecha de cambio de estado y el precio de venta aplicado.

Finalmente, la analítica comercial no genera ingreso directo pero optimiza la adquisición de catálogo y la asignación de recursos. El sistema debe agregar montos de pagos por dimensiones temáticas, geográficas y temporales, preservando la granularidad transaccional para consultas ad-hoc.

Con esta comprensión del modelo de negocio, podemos ahora identificar quiénes participan en la operación y qué responsabilidades asume cada uno.

### 2.4. Estructura Organizacional y Actores Clave: Quiénes Hacen Funcionar el Negocio

#### 2.4.1. Los Actores y sus Roles

El cliente es el usuario final que accede al servicio de renta. Se registra en una tienda, selecciona títulos, completa rentas y devoluciones, y consulta su historial. Para gestionarlo, el sistema requiere identificación personal básica, medio de contacto, domicilio, tienda de adscripción y estado de cuenta.

El empleado operativo atiende en mostrador y procesa transacciones. Registra rentas, cobros y devoluciones; da de alta clientes; consulta catálogo y disponibilidad. El sistema gestiona sus credenciales de acceso únicas, identificación personal, domicilio, establecimiento de adscripción y estado laboral.

El administrador de tienda es el responsable operativo y administrativo de una sede. Supervisa inventario, valida reportes, asigna turnos y resuelve incidencias comerciales. El sistema lo vincula de manera exclusiva con el establecimiento que administra y le asigna permisos diferenciados para funciones de supervisión.

El gestor de catálogo, aunque implícito en la operación cotidiana, es el curador centralizado del contenido disponible. Da de alta y mantiene títulos, participantes, categorías, idiomas y tarifas; negocia con productoras. El sistema le otorga acceso a funciones de administración de catálogos maestros, sin participación en transacciones operativas con clientes.

Finalmente, el sistema de información actúa como infraestructura que soporta la toma de decisiones. Registra estadísticas de rentas, controla inventario en tiempo real y genera reportes analíticos. Sus propios requisitos de desempeño, disponibilidad y trazabilidad justifican su desarrollo e implementación.

#### 2.4.2. Cómo se Relacionan estos Actores

La operación del negocio se sostiene en relaciones claras entre sus participantes. Un cliente se registra obligatoriamente en un establecimiento comercial, lo que permite personalizar el servicio y segmentar análisis por ubicación geográfica. Un establecimiento emplea a múltiples colaboradores operativos, pero designa a un único responsable administrativo que no puede ejercer esa función en otra sede al mismo tiempo.

El responsable administrativo supervisa el inventario físico y valida reportes, mientras que los colaboradores operativos ejecutan las transacciones cotidianas de renta, cobro y devolución. El gestor de catálogo mantiene de manera centralizada la información maestra sobre títulos, participantes y clasificaciones, la cual se distribuye conceptualmente a los inventarios locales de cada establecimiento.

Estas relaciones no son meras descripciones organizacionales; cada una implica necesidades específicas de registro, validación y trazabilidad que el sistema deberá atender. Veamos ahora cómo se materializan en los procesos operativos centrales.

### 2.5. Procesos Operativos Centrales: El Ritmo Cotidiano del Negocio

#### 2.5.1. Administrar el Catálogo Maestro

El negocio necesita mantener un registro estructurado de cada título audiovisual disponible para renta. Esto incluye información descriptiva como nombre, sinopsis, año de estreno y duración; clasificación etaria normativa; tarifas comerciales; y soporte para múltiples idiomas, tanto la versión comercial como, opcionalmente, el idioma original de producción.

Para soportar este proceso, el sistema debe identificar de manera única e invariante cada título registrado; almacenar descripciones textuales extensas y permitir búsquedas semánticas sobre ese contenido; asociar cada título con múltiples categorías temáticas, evitando vínculos duplicados; registrar participantes del elenco, permitiendo que una misma persona aparezca en múltiples títulos y viceversa, sin duplicar la asociación; y mantener catálogos maestros de idiomas y categorías con denominaciones únicas, para evitar inconsistencias en la clasificación.

#### 2.5.2. Controlar el Inventario Físico por Sucursal

Cada ejemplar físico de un título debe ser distinguible individualmente y asignarse explícitamente a un establecimiento específico. El sistema debe permitir consultar, en tiempo real, cuántos ejemplares de un título existen en cada sede y cuáles están disponibles para renta en un momento dado.

Para lograrlo, el sistema asigna un identificador único a cada ejemplar físico, independiente del título al que pertenece; vincula explícitamente cada ejemplar con el establecimiento donde se encuentra físicamente; verifica, al procesar una renta, que el ejemplar solicitado no esté ya asignado a otra transacción activa; y registra cuándo fue la última modificación de cada registro de inventario, para fines de auditoría y sincronización.

#### 2.5.3. Procesar una Renta y su Devolución en Mostrador

Cuando un cliente solicita rentar un ejemplar disponible, el empleado operativo registra la transacción capturando la fecha y hora exactas, el ejemplar seleccionado, la identificación del cliente y su propia credencial como procesador. La fecha de devolución se registra después, cuando el cliente retorna el ejemplar; si no hay registro de devolución, se entiende que el ejemplar sigue en posesión del cliente.

El sistema responde registrando obligatoriamente fecha y hora de inicio, ejemplar rentado, cliente y empleado procesador en cada transacción de renta; impidiendo que un mismo ejemplar sea asignado a múltiples transacciones en el mismo instante temporal; permitiendo registrar opcionalmente la fecha y hora de devolución, para consultar qué ejemplares están pendientes de retorno; y preservando históricamente la transacción completa, incluso si el cliente o empleado asociado cambia de estado operativo después.

#### 2.5.4. Procesar Pagos y Mantener Auditoría Transaccional

Cada transacción de renta genera uno o más registros de cobro. Estos deben documentar el monto monetario, la fecha exacta del procesamiento, el cliente que realiza el pago y el empleado que lo procesa. Todo cobro debe estar vinculado explícitamente a una renta válida.

El sistema garantiza esta integridad vinculando obligatoriamente cada registro de cobro con la transacción de renta que lo origina; validando que los montos registrados sean estrictamente positivos y consistentes con las tarifas vigentes; registrando fecha y hora exactas para cada cobro, permitiendo reportes financieros por período; y preservando la integridad referencial: si un cliente o empleado se desactiva, sus cobros históricamente asociados no deben eliminarse ni invalidarse.

#### 2.5.5. Normalizar Información Geográfica y Datos Maestros

Las direcciones físicas de clientes, empleados y establecimientos deben estructurarse jerárquicamente: país, ciudad, distrito, calle. Esta normalización evita redundancia y facilita mantenimiento centralizado. Si cambia la denominación de una ciudad, por ejemplo, ese cambio debe reflejarse en todos los registros que la referencian.

El sistema implementa esta estrategia separando países, ciudades y direcciones en registros independientes, vinculados mediante referencias explícitas; permitiendo que un mismo registro de dirección sea compartido por múltiples actores sin duplicar información; e impidiendo la eliminación de un país mientras existan ciudades registradas que lo referencien, y análogamente para ciudades y direcciones.

Estos procesos no operan en el vacío; están condicionados por reglas de negocio que el sistema debe respetar de manera estricta.

### 2.6. Reglas de Negocio: Los Límites que Dan Forma al Sistema

El negocio opera dentro de un marco de reglas que condicionan el diseño del sistema. La clasificación etaria normativa exige que los contenidos se clasifiquen según un conjunto predefinido de categorías: apto para todo público, adolescentes, adultos, entre otros. El sistema responde validando que ningún registro de título utilice un valor de clasificación fuera del catálogo autorizado.

Por política empresarial explícita, ningún establecimiento puede comercializar contenido clasificado como exclusivo para adultos. El sistema impide el registro o la exhibición de títulos que violen esta restricción, mediante validación de dominio.

La unicidad en la asociación elenco-título garantiza que un participante del elenco no pueda aparecer más de una vez asociado al mismo título. El sistema asegura esta regla garantizando que la combinación de participante y título sea única en el registro de asociaciones.

La disponibilidad exclusiva por ejemplar establece que un ejemplar físico no puede ser rentado por múltiples clientes en el mismo instante. El sistema implementa un mecanismo de bloqueo lógico que previene asignaciones concurrentes sobre el mismo ejemplar.

La vinculación financiera obligatoria exige que todo registro de cobro esté asociado explícitamente a una transacción de renta válida. El sistema valida la existencia de la renta referenciada antes de permitir registrar un cobro.

La preservación del historial transaccional establece que desactivar un cliente o empleado no debe eliminar ni alterar los registros históricos de rentas y cobros asociados. El sistema implementa una estrategia de desactivación lógica que preserve la integridad referencial del historial.

Finalmente, la auditoría temporal uniforme exige que todos los registros maestros y transaccionales documenten cuándo fueron modificados por última vez. El sistema registra automáticamente una marca temporal de actualización en cada operación de modificación.

Estas reglas no son sugerencias; son condiciones no negociables que el sistema debe incorporar desde su diseño fundamental.

### 2.7. Toma de Decisiones: Información que Ilumina la Acción

#### 2.7.1. Reportes Operativos Básicos

Para el día a día de la operación, el sistema debe permitir consultar la disponibilidad de inventario por título y sede: para un título específico, ¿cuántos ejemplares hay en cada establecimiento y cuántos están disponibles ahora mismo? También debe ofrecer el historial de rentas por cliente: un listado cronológico de todas las transacciones de un cliente, con fechas, ejemplares, montos y estado de devolución. Y finalmente, el directorio de personal por sede: qué empleados operativos y administrativos están asignados a cada establecimiento, con sus datos de contacto y estado laboral.

#### 2.7.2. Reportes Analíticos para la Gestión Comercial

Para la toma de decisiones estratégicas, se requieren capacidades de agregación y análisis. Los ingresos por categoría temática y período responden a la pregunta: ¿qué géneros generan más ingresos en un rango de fechas determinado? El desempeño comparado por establecimiento permite evaluar: ¿cómo se comparan las sedes en términos de ingresos, número de transacciones y rotación de inventario? Y las tendencias de popularidad identifican: ¿qué títulos, participantes o categorías están siendo más rentados en períodos específicos, para ajustar estrategias de exhibición?

#### 2.7.3. Calidad de la Información: La Confianza como Requisito Transversal

Más allá de qué información se registra, importa cómo se garantiza su confiabilidad. La consistencia exige que si cambia la denominación de una ciudad en el catálogo maestro, ese cambio se refleje automáticamente en todos los registros que la referencian. La integridad requiere que las relaciones entre registros, por ejemplo un cobro vinculado a una renta, se preserven mediante validaciones que impidan referencias a registros inexistentes.

La trazabilidad demanda que cada modificación en registros críticos quede documentada con una marca temporal que permita auditoría retrospectiva. Y el desempeño operativo establece que las consultas de disponibilidad de inventario y el registro de transacciones deben completarse en tiempo aceptable, dos segundos o menos, bajo carga concurrente nominal.

### 2.8. El Sistema de Información: Condiciones que Moldean el Diseño

El sistema no opera en un vacío técnico; debe respetar condiciones organizacionales, normativas y estratégicas. La disponibilidad del servicio exige operación continua veinticuatro horas al día, siete días a la semana, trescientos sesenta y cinco días al año, tolerando fallos parciales sin comprometer transacciones en curso. Esto garantiza servicio ininterrumpido para clientes y empleados en múltiples zonas horarias, e implica un diseño que garantice atomicidad de transacciones, recuperación ante fallos y mecanismos de concurrencia seguros.

La arquitectura de referencia adopta un modelo centralizado o federado por sedes, con separación lógica entre presentación, lógica de negocio y gestión de datos. Esto facilita mantenimiento, control de accesos y escalabilidad independiente por capa, e implica una estructura de datos normalizada, interfaces claras entre componentes y documentación de dependencias.

La soberanía digital establece que los datos personales, transaccionales y de auditoría deben residir bajo jurisdicción nacional o acuerdos explícitos de gobernanza. Esto responde al cumplimiento normativo local, protección de privacidad y control administrativo, e implica normalización de datos personales, registro de auditoría temporal y control de accesos basado en roles.

La escalabilidad económica exige que la estructura permita incorporar nuevas sedes, expandir catálogo y acumular historial sin reestructuración de fondo. Esto optimiza inversión inicial y garantiza viabilidad de crecimiento orgánico, e implica diseño normalizado que evite redundancias, identificadores estables y estrategias de particionamiento para volúmenes crecientes.

Finalmente, la protección de datos establece que la información personal de clientes y empleados debe manejarse con medidas de seguridad apropiadas. Esto cumple principios de privacidad y minimización de datos, e implica capacidad para gestionar estados lógicos de activación y desactivación, así como técnicas de anonimización controlada.

### 2.9. Visión Estratégica: Hacia Dónde Podría Evolucionar este Negocio

En el corto plazo, de cero a doce meses, el negocio busca estabilizar la operación en todas las sedes activas con un catálogo inicial de aproximadamente mil títulos; enriquecer continuamente el catálogo mediante acuerdos con productoras de serie B y contenido directo para video; e implementar métricas básicas de rotación de inventario por tienda y categoría para orientar decisiones de adquisición.

En el mediano plazo, de doce a treinta y seis meses, aspira a coordinar logística inter-sedes para transferir ejemplares y optimizar la disponibilidad regional; expandir capacidades analíticas para predecir demanda estacional, identificar tendencias de género y adquirir títulos estratégicamente; y formalizar programas de retención basados en historial transaccional y preferencias de clasificación temática.

En el largo plazo, más allá de treinta y seis meses, evalúa la integración con canales de acceso digital como reserva en línea y notificaciones de disponibilidad; implementa modelos de suscripción o membresía recurrente con beneficios diferenciados; y moderniza la infraestructura de información para soportar inteligencia comercial avanzada y gobernanza automatizada.

Esta visión no es un plan ejecutivo; es un ejercicio de prospectiva que ayuda a entender cómo las decisiones de diseño actuales pueden facilitar o limitar la evolución futura del sistema.

### 2.10. Conexión con el Contexto: De la Conceptualización a la Implementación

Este documento conceptual no viaja solo. Para garantizar que los términos, reglas implícitas y restricciones operativas aquí descritas no queden sujetas a interpretaciones ambiguas, se ha incorporado como Anexo A el Documento de Vocabulario y Reinterpretación Semántica. Este anexo funciona como un glosario operativo y un puente de precisión: traduce el lenguaje natural del negocio a una formalización semántica lista para ser transformada en requisitos técnicos. Al integrarlo directamente en esta parte, aseguramos que la base conceptual y su formalización terminológica se lean de manera conjunta y coherente.

Una vez consolidada esta base, el estudio de caso avanza hacia las fases tradicionales de ingeniería de software. La especificación de requerimientos, conforme a estándares IEEE e ISO, traduce los procesos, actores y reglas formalizadas en el Anexo A a requisitos funcionales, de datos y no funcionales verificables. El diseño lógico estructura esos requisitos en un modelo conceptual independiente de tecnología, aplicando normalización sin atarse a un motor específico. El diseño físico materializa el modelo lógico en objetos técnicos concretos, aprovechando las capacidades del motor de bases de datos seleccionado. Y la validación final cierra el ciclo metodológico verificando que el esquema regenerado corresponda punto por punto con el artefacto de referencia original.

La coherencia entre esta descripción del negocio, su anexo semántico y los artefactos técnicos posteriores es lo que garantiza que este estudio de caso no sea un ejercicio de simulación aislado, sino una práctica rigurosa y trazable de análisis y diseño de sistemas.

### 2.11. Control de Versiones

| Versión | Fecha | Autor | Cambios Principales |
|---------|-------|-------|---------------------|
| 1.0 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Formalización inicial del vocabulario, reglas implícitas y patrones de asociación del modelo operativo. |
| 1.1 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Ajuste de terminología para alineación con Parte 2, eliminación de referencias técnicas prematuras y reforzamiento de trazabilidad hacia SRS. |


---

## Anexo 2.A: Documento de Vocabulario del Negocio de Renta de Películas

### 2.A.1. Propósito y Alcance

Este anexo tiene como finalidad **formalizar el lenguaje operativo del negocio** descrito en la Parte 2, traduciéndolo de su expresión natural a una definición semántica rigurosa y desambiguada. Su propósito no es técnico ni prescriptivo en términos de implementación, sino **conceptual y de alineación**: garantizar que cada término, regla implícita y restricción mencionada en el modelo operativo tenga una interpretación única, verificable y lista para ser transformada en requisitos de sistema.

El documento funciona como un **puente semántico** entre la descripción del negocio y las fases de ingeniería de requisitos y diseño. Al integrar la terminología, las relaciones conceptuales y las restricciones abstractas en un solo artefacto, se evita la ambigüedad interpretativa y se establece una base común para analistas, diseñadores y evaluadores académicos.

**Alcance:**
- Definición formal de los conceptos nucleares del dominio.
- Explicitación de reglas de negocio implícitas en la operación cotidiana.
- Establecimiento de restricciones abstractas independientes de tecnología.
- Vinculación directa con las necesidades de información que derivarán en requisitos funcionales y no funcionales.

**Exclusiones:**
- Referencias a estructuras de datos, lenguajes de consulta o motores de almacenamiento.
- Especificación de algoritmos, flujos de código o arquitecturas de software.
- Documentación de interfaces de usuario o prototipos interactivos.

### 2.A.2. Vocabulario de Negocio

La siguiente matriz sistematiza la terminología utilizada por los actores del negocio, su formalización conceptual por parte del análisis de sistemas, la regla operativa que la sustenta y la necesidad de información que el sistema debe satisfacer.

| Término del Negocio | Definición Formal | Regla de Negocio Explícita | Necesidad de Información para el Sistema |
|---------------------|----------------------------|----------------------------|------------------------------------------|
| **Título** | Obra audiovisual identificable de manera única, que contiene metadatos descriptivos, clasificación etaria, tarifas y soporte para múltiples idiomas. | Todo título debe poseer un identificador invariante y al menos una clasificación etaria dentro del catálogo autorizado. | Registrar nombre, sinopsis, año, duración, tarifas, idioma de versión y opcionalmente idioma original. |
| **Catálogo Maestro** | Conjunto centralizado y normalizado de títulos, participantes, categorías e idiomas que sirve como referencia única para toda la red de establecimientos. | Los elementos del catálogo deben mantenerse coherentes; un cambio en un registro maestro se refleja transversalmente en todas las referencias. | Mantener registros independientes y vinculados para títulos, categorías, idiomas y participantes. |
| **Participante / Elenco** | Persona natural que interviene en la producción de un título audiovisual. | Un título puede asociarse a múltiples participantes y viceversa; la asociación debe ser única y no duplicarse. | Registrar nombre y apellido; permitir asociaciones múltiples sin repetición en la misma obra. |
| **Categoría Temática** | Clasificación narrativa, estilística o comercial que agrupa títulos con afinidad de contenido. | Un título puede pertenecer a múltiples categorías; las denominaciones de categoría deben ser únicas. | Permitir asociaciones flexibles muchos-a-muchos y validar unicidad nominal. |
| **Ejemplar / Copia Física** | Unidad material distinguible de un título, asignada explícitamente a un establecimiento para su comercialización temporal. | Cada ejemplar requiere identificación propia; su disponibilidad depende exclusivamente del estado de las transacciones activas que lo involucren. | Identificar de manera única cada unidad, vincularla a un establecimiento y registrar su estado de ocupación en tiempo real. |
| **Renta / Préstamo Comercial** | Transacción que otorga posesión temporal de un ejemplar a un cliente, documentando fecha de inicio, responsable operativo y fecha de devolución. | Debe registrar obligatoriamente ejemplar, cliente, fecha y empleado procesador. Impide la concurrencia sobre el mismo ejemplar en el mismo instante. | Capturar fecha/hora de inicio, ejemplar, cliente y empleado; permitir fecha de devolución opcional; validar exclusividad temporal. |
| **Devolución** | Evento posterior que cierra el ciclo de renta, registrando la fecha y hora en que el ejemplar regresa al establecimiento. | La ausencia de registro de devolución indica posesión activa por parte del cliente. | Permitir actualización de fecha de devolución; mantener historial completo incluso tras el cierre. |
| **Pago / Cobro** | Registro financiero vinculado obligatoriamente a una renta, que documenta monto, fecha exacta y empleado procesador. | Todo cobro debe referir una renta válida; los montos deben ser estrictamente positivos. | Vincular explícitamente a la renta, registrar monto y fecha, validar no negatividad y preservar ante cambios de estado de clientes/empleados. |
| **Cliente** | Persona natural registrada que accede al servicio bajo los términos comerciales de la cadena. | Se asocia a un establecimiento de origen; su estado operativo es lógico y no elimina su historial transaccional. | Registrar identidad básica, contacto, domicilio, tienda de adscripción y estado (activo/inactivo). |
| **Empleado Operativo** | Personal adscrito a un establecimiento con credenciales para registrar y validar transacciones comerciales. | Debe contar con identificación, domicilio y establecimiento de adscripción; el nombre de usuario debe ser único. | Gestionar credenciales únicas, datos personales, tienda asignada y estado laboral. |
| **Administrador de Tienda** | Responsable operativo y administrativo de un establecimiento, con funciones de supervisión y validación. | Único por establecimiento; no puede ejercer administración en múltiples sedes simultáneamente. | Vincular de manera exclusiva a un establecimiento y asignar permisos diferenciados de supervisión. |
| **Establecimiento / Tienda** | Ubicación física operativa donde se gestiona inventario local, se registra a clientes y se procesan transacciones. | Requiere dirección física válida y un único responsable administrativo asignado. | Registrar ubicación, responsable único y datos de contacto operativos. |
| **Dirección / Ubicación Física** | Referencia geográfica y postal normalizada que describe la localización de clientes, empleados y establecimientos. | Se estructura jerárquicamente (país → ciudad → distrito → calle) y se comparte entre múltiples actores para evitar redundancia. | Normalizar en registros independientes y vinculados; impedir eliminación de registros superiores si existen dependencias activas. |
| **Clasificación Etaria** | Normativa que restringe el acceso a contenidos según grupos de edad. | Debe pertenecer exclusivamente al conjunto predefinido autorizado; se prohíbe explícitamente contenido para adultos. | Validar contra un catálogo cerrado de valores; rechazar registros fuera del dominio permitido. |
| **Estado de Cuenta / Estado Operativo** | Indicador lógico que determina si un cliente o empleado puede participar activamente en transacciones. | La desactivación no elimina, altera ni invalida el historial transaccional asociado. | Implementar bandera lógica de activación/desactivación con preservación de integridad referencial histórica. |
| **Marca Temporal de Auditoría** | Registro del instante exacto en que un dato maestro o transaccional fue modificado por última vez. | Se aplica de manera uniforme para garantizar trazabilidad temporal y sincronización de estados. | Actualizar automáticamente en cada operación de modificación; no identificar al autor, solo certificar la ocurrencia temporal. |

### 2.A.3. Relaciones Lógicas y Patrones de Asociación

La operación del negocio se sostiene en un entramado de relaciones semánticas que el sistema debe modelar con precisión. A continuación, se describen los patrones conceptuales clave:

| Patrón | Significado | Implicación para el Modelado |
|-------------------|----------------------|-----------------------------|
| **Asociación Muchos-a-Muchos (M:N)** | Un título puede vincularse a múltiples categorías y participantes; inversamente, una categoría o participante puede asociarse a múltiples títulos. | Requiere una estructura intermedia que resuelva la asociación sin duplicidad, garantizando unicidad en la combinación de identificadores. |
| **Jerarquía Geográfica (N:1 encadenada)** | Una dirección pertenece a una ciudad; una ciudad pertenece a un país. La normalización evita redundancia y facilita mantenimiento centralizado. | Se modela mediante referencias explícitas hacia arriba en la jerarquía, con restricciones que impidan la eliminación de niveles superiores mientras existan dependencias activas. |
| **Asignación Exclusiva (1:1)** | Un establecimiento designa un único administrador; un empleado no puede administrar más de un establecimiento a la vez. | Requiere una restricción de unicidad sobre la referencia al administrador, garantizando correspondencia biunívoca. |
| **Vinculación Transaccional Obligatoria (N:1)** | Cada pago y cada renta deben referir obligatoriamente a un cliente y a un establecimiento válidos. | Se modela mediante referencias explícitas que validan la existencia del registro origen antes de permitir la creación del transaccional. |
| **Bloqueo Temporal de Disponibilidad** | Un ejemplar no puede estar rentado por múltiples clientes en el mismo instante. | El sistema debe validar la no superposición de intervalos temporales o instantes de registro para el mismo ejemplar. |

### 2.A.4. Restricciones y Contexto

Más allá de las reglas explícitas, el negocio opera bajo condiciones de contexto que condicionan el diseño del sistema. Estas restricciones se formulan de manera abstracta, sin vinculación a tecnologías específicas:

| Dimensión | Restricción Conceptual | Fundamento Operativo | Traducción a Requisito de Sistema |
|-----------|----------------------|---------------------|----------------------------------|
| **Disponibilidad** | Operación continua 24×7×365, tolerando fallos parciales sin comprometer transacciones en curso. | Garantizar servicio ininterrumpido en múltiples zonas horarias. | Transacciones atómicas, mecanismos de concurrencia seguros y recuperación ante fallos. |
| **Integridad Histórica** | La desactivación de clientes o empleados no elimina ni invalida transacciones previas. | Cumplimiento legal, auditoría y análisis de tendencias. | Estrategia de desactivación lógica con preservación de referencias históricas. |
| **Consistencia Transversal** | Los cambios en catálogos maestros se reflejan automáticamente en todos los registros dependientes. | Evitar obsolescencia y mantener coherencia operativa. | Referencias explícitas y propagación controlada de actualizaciones. |
| **Soberanía y Privacidad** | Los datos personales y transaccionales residen bajo jurisdicción y gobernanza explícita. | Cumplimiento normativo y protección de la información. | Control de accesos basado en roles, anonimización para entornos de prueba y trazabilidad de auditoría. |
| **Desempeño Operativo** | Consultas de disponibilidad y registro de transacciones en ≤ 2 segundos bajo carga concurrente nominal. | Mantener fluidez en mostrador y evitar cuellos de botella. | Estructura optimizada para búsquedas frecuentes y validaciones en tiempo real. |

### 2.A.5. Trazabilidad

Este anexo no es un documento terminal; es un **artefacto de transición**. Su contenido se transforma directamente en los siguientes entregables del ciclo de desarrollo:

| Elemento de este Anexo | Derivación en la Fase Siguiente |
|------------------------|--------------------------------|
| Términos y definiciones formales | Base para el glosario de la SRS y nombres de módulos funcionales. |
| Reglas de negocio explícitas | Requisitos funcionales verificables (RF) con criterios de aceptación medibles. |
| Patrones de asociación (M:N, 1:1, N:1, jerarquías) | Diseño del modelo Entidad-Relación lógico y patrones de mapeo relacional. |
| Restricciones abstractas y de contexto | Requisitos no funcionales (RNF) de disponibilidad, desempeño, integridad y seguridad. |
| Necesidades de información | Requisitos de datos (RD) y especificación de vistas analíticas y operativas. |

La coherencia entre esta formalización semántica y la especificación técnica que le sigue garantiza que cada decisión de diseño pueda rastrearse hasta una necesidad real del negocio, cumpliendo con los estándares de ingeniería de requisitos (IEEE/ISO) y los objetivos pedagógicos de la UEA.

---

> **Declaración de Propósito Académico:**  
> Este anexo ha sido elaborado exclusivamente con fines pedagógicos dentro del estudio de caso de la UEA Bases de Datos (2151106) de la Universidad Autónoma Metropolitana, Unidad Iztapalapa. Su contenido formaliza hipotéticamente el vocabulario y las reglas semánticas que sustentan el modelo operativo descrito en la Parte 2, integrando principios de ingeniería de requisitos y análisis conceptual. No constituye un documento contractual ni prescribe tecnologías de implementación.

---

**Elaborado para distribución educativa**  
**Universidad Autónoma Metropolitana - Unidad Iztapalapa**  
*Licenciatura en Computación | UEA Bases de Datos (2151106)*  
*Documento de distribución educativa bajo licencia CC BY-NC-SA 4.0. Se autoriza la reproducción parcial con citación institucional completa.*