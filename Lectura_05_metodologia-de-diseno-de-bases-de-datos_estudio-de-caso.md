### Lectura 05. Metodología Profesional de Diseño de Bases de Datos
### Estudio de Caso: Sistema de Registro Académico Universitario
(draft)

**Dr. Jesús Zavala Ruiz**  
(Junio de 2026)

---

#### Introducción

El **diseño de bases de datos** constituye, en mi opinión, mucho más que un mero ejercicio técnico de almacenamiento de información; representa la columna vertebral conceptual de cualquier aplicación moderna. Una estructura bien concebida no solo resguarda datos, sino que garantiza la integridad, la eficiencia operativa y la escalabilidad del sistema a lo largo del tiempo, formalizando parcialmente el conocimiento organizacional (el *orgware*, por así decirlo). En este ámbito, la metodología de diseño de bases de datos sistematizada por Fidel A. Captain (2015), como los Seis Pasos, se presenta no como una simple receta o un *framework* de moda (*management fad*), sino como una *práctica* rigurosa que ofrece un enfoque sistemático para transformar los requerimientos empresariales —esa construcción social de la realidad operativa— en un modelo relacional sólido.

Esta metodología guía al diseñador de bases de datos mediante un proceso que, desde la lógica de Charles S. Peirce, podríamos calificar de abductivo: partimos de los "hechos sorprendentes" y las anomalías de los reportes actuales del cliente, para sugerir una hipótesis plausible que se materializa en el diagrama de Modelo Relacional (R-M). El proceso de normalización actúa como el marco teórico indispensable que guía el diseño de las bases de datos relacionales. La normalización no es solo una regla técnica; es el mecanismo de defensa contra la fragmentación lógica, asegurando que la visión del negocio se refleje con precisión y sin las redundancias que, de otro modo, convertirían a la base de datos en un auténtico Frankenstein.

Dicho de otra manera, esta metodología estructura el **proceso de diseño de bases de datos** en tres fases fundamentales que bien podríamos calificar como un tránsito metodológico. La **primera fase** de **descubrimiento de identidades** (Pasos 1 y 2) se centra en la identidad del dominio: identifica las entidades de interés, asigna sus atributos atómicos y establece las relaciones unarias y binarias mediante una Matriz Entidad-Entidad. La **segunda fase**, de **modelado conceptual** (Pasos 3, 4 y 5), traduce estos hallazgos en un diagrama Entidad-Relación (E-R) simplificado que, tras la enumeración exhaustiva de afirmaciones de opcionalidad y cardinalidad, culmina en un *diagrama E-R* detallado. Este modelo conceptual captura fielmente *la visión del usuario y las reglas de negocio* institucionales. Finalmente, la **tercera fase** de **implementación lógica** (Paso 6) transforma el modelo conceptual en un *diagrama del Modelo Relacional* (M-R) con notación de Pata de Gallo (*Crow's Foot*), resolviendo las relaciones complejas mediante tablas de unión y llaves foráneas para obtener un *esquema normalizado* y listo para su despliegue en cualquier Sistema Manejador de Bases de Datos Relacionales (RDBMS).

El estudio de caso, retomado de Captain (2015), aborda el diseño de una base de datos para el departamento de registro de una universidad de tamaño mediano. La problemática central radica en la necesidad de gestionar, de manera integral, la programación semestral de clases, la asignación de docentes, la inscripción estudiantil y el control de accesos. Sin embargo, un aspecto fundamental que deseo destacar —y que a menudo se pasa por alto en los enfoques puramente tecnicistas— es la clara distinción entre *dos perfiles de interacción* con el sistema de información: los usuarios operativos y los usuarios de negocio.

Los **usuarios operativos** corresponden al personal administrativo y técnico responsable de configurar, mantener y supervisar el funcionamiento del sistema. Su interacción se centra en la gestión de la infraestructura lógica, la administración de credenciales, la asignación de niveles de autorización y la supervisión o auditoría de la actividad del sistema. Por otro lado, los **usuarios de negocio** comprenden a estudiantes, personal académico y administrativo que interactúan directamente con la funcionalidad académica: consultar horarios, realizar inscripciones o gestionar su perfil curricular.

Por lo anterior, es plausible afirmar que esta separación conceptual permite estructurar la **arquitectura de la base de datos** de manera que refleje fielmente los distintos niveles de responsabilidad y acceso, evitando lo que en teoría de la organización llamaríamos "confusión de roles". A través de los Seis Pasos de la metodología de Captain, se modelará cada componente de manera independiente pero cohesiva, asegurando que las reglas de negocio, los flujos de acceso y los requisitos técnicos converjan en un diseño preciso, normalizado y listo para su implementación en producción, cumpliendo los objetivos organizacionales.

#### Fase I: Descubrimiento de Entidades (Pasos 1 y 2)

Antes de dibujar tablas, es imperativo comprender la naturaleza misma del negocio. Esta fase constituye un ejercicio de abducción: extraer del caos aparente de los reportes cotidianos los objetos tangibles que la organización "desea llevar el control de", para posteriormente mapear la praxeología de sus interacciones mediante la Matriz Entidad-Entidad.

#### Paso 1: Descubrimiento de entidades y atributos

En mi opinión, el diseño de una base de datos relacional no comienza jamás dibujando tablas, sino **comprendiendo la naturaleza misma del negocio**. Como señala acertadamente Fidel A. Captain: «Se hace gran énfasis en esta etapa del proceso porque es la más importante y porque los errores cometidos durante esta etapa se propagarán a través de todas las demás etapas y se reflejarán en la base de datos real» (Captain, 2015, p. 16). Dicho de otra manera, los errores conceptuales cometidos en esta fase se cristalizarán irremediablemente en la estructura física del sistema.

El objetivo de este paso es sencillo pero crítico desde una perspectiva metodológica: *traducir la narrativa del cliente* —esa construcción social de su realidad operativa— en un catálogo ordenado de **qué vamos a rastrear** (las **entidades**) y cómo lo describiremos (los **atributos**). Si esta base conceptual es sólida, los pasos siguientes fluirán con precisión; si es ambigua, arrastraremos inconsistencias hasta la implementación física, creando lo que en la jerga de los sistemas legacy podríamos llamar un auténtico Frankenstein, un *spaghetti* tecnológico que, al igual que la criatura de Mary Shelley, terminará por volverse contra sus propios creadores.

##### 1.0. Especificación del Problema

El jefe de registro académico de una pequeña universidad desea una aplicación que ayude a su departamento a llevar el control de las *clases programadas*, los *cursos* y *docentes* que aparecen en el horario de los cursos y los *estudiantes* que se inscriben en cursos, de acuerdo con el horario. Los cursos se programan cada semestre y esto se documenta en el horario de clases, el cual también documenta los docentes asignados a cada clase programada. Los *estudiantes* se inscriben en cursos de acuerdo con la lista de clases programadas. Los *usuarios* (estudiantes, personal docente y administrativo) deben iniciar sesión en la aplicación para obtener acceso y la aplicación debe llevar el control de las entradas y salidas de los usuarios, así como de las modificaciones a los registros. Además, los usuarios deben tener diferentes niveles de acceso, lo que determinará su acceso a diferentes partes de la aplicación.

Este **planteamiento del problema** cumple con los criterios establecidos por Captain (2015, p. 16) para un **enunciado claro y conciso**. Aquí es imperativo destacar que la técnica de rastrear la frase **"llevar el control de"** no es una mera heurística improvisada, sino un ***tip*** **metodológico explícito**, sistematizado por el propio F. A. Captain. En mi opinión, esta directriz actúa como una brújula metodológica: la frase clave **"desea llevar el control de"** orienta directamente y sin ambigüedades el proceso abductivo de descubrimiento de entidades, delimitando el dominio funcional y estableciendo los requisitos operativos esenciales.

Para fundamentar esta necesidad y comprender el contexto real en el que opera la universidad, el cliente proporciona extractos de sus reportes en hojas de cálculo. Estos reportes funcionan como insumos brutos, artefactos que ejemplifican la organización actual de sus datos y complementan el contexto del problema, ofreciendo una aproximación más realista al diseño práctico.

**Insumo A: Ejemplo del Reporte de Programación Escolar**

| Periodo | Curso | Docente Asignado | Horario | Ubicación |
| :--- | :--- | :--- | :--- | :--- |
| Semestre Año-1 (2026-1) | Cálculo I (MAT101) | Dr. Roberto Sánchez García | Lunes 10:00-12:00 | Aula 304 |
| Semestre 2 (2026-2) | Programación I (CS101) | Mtra. Diana Torres | Miércoles 14:00-16:00 | Lab. Cómputo 2 |

**Insumo B: Reporte de Lista de Grupo**

| Nombre del Estudiante | Correo Estudiante | Curso (Materia) | Nombre del Profesor | Correo Profesor | Día y Hora | Aula |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Ana García López | ana.garcia@uni.edu | Cálculo I (MAT101) | Dr. Roberto Sánchez García | r.sanchez@uni.edu | Lunes 10:00-12:00 | Aula 304 |
| Carlos Méndez Ruiz | carlos.mendez@uni.edu | Cálculo I (MAT101) | Dr. Roberto Sánchez García | r.sanchez@uni.edu | Lunes 10:00-12:00 | Aula 304 |
| Ana García López | ana.garcia@uni.edu | Programación I (CS101) | Mtra. Diana Torres | d.torres@uni.edu | Miércoles 14:00-16:00 | Lab. Cómputo 2 |
| Luis Fernández | luis.fernandez@uni.edu | Cálculo I (MAT101) | Dr. Roberto Sánchez García | r.sanchez@uni.edu | Lunes 10:00-12:00 | Aula 304 |

Al observar estos reportes, es evidente que la organización actual, particularmente en la "Lista de Grupo" (Insumo B), agrupa múltiples piezas de información en una sola tabla plana. Si bien esto es funcional para una lectura humana rápida o para imprimir un acta de calificaciones, desde la perspectiva de un sistema de información, esta agrupación oculta la complejidad de los datos y viola los principios más básicos de la organización eficiente de los datos. Por ejemplo, los datos del "Dr. Roberto Sánchez García" y su correo, así como el "Lunes 10:00-12:00" y el "Aula 304", se repiten mecánicamente por cada estudiante inscrito. Del mismo modo, el nombre del curso "Cálculo I (MAT101)" está fusionado con los datos de la clase. Esta redundancia no solo desperdicia almacenamiento, sino que invita a la temida anomalía de actualización: si el Dr. Sánchez cambia su correo, habría que actualizar decenas de filas en las listas de grupo, arriesgando la consistencia de los datos.

##### 1.1 Iteración 1: La visión cruda del negocio

El primer filtro consiste en extraer del planteamiento del problema y de los insumos los **sustantivos** que representan objetos tangibles, persistentes y de interés directo. Por convención, los nombramos en plural para facilitar el manejo de relaciones en etapas posteriores.

Tras analizar los requisitos y los reportes, identificamos siete **objetos** o **entidades de interés** claramente delimitados: `estudiantes`, `cursos`, `clases_programadas`, `docentes`, `usuarios`, `niveles_acceso` y `registros_actividad`.

En esta primera iteración y, alineados con la visión inicial del cliente reflejada en los reportes, listamos los **atributos** de las entidades, tal como se perciben en un primer acercamiento:

- **estudiantes:** `nombre_completo`, `correo_electronico`, `telefono`, `direccion` y `fecha_nacimiento`.  
- **cursos:** `codigo_curso`, `nombre_corto`, `nombre_largo` y `descripcion_curso`.  
- **clases_programadas:** `codigo_horario`, `seccion`, `dia`, `hora`, `ubicacion`, `periodo_o_semestre`, `nombre_profesor` y `lista_estudiantes`.  
- **docentes:** `nombre_completo`, `correo_electronico`, `telefono`, `direccion` y `acerca_de`.  
- **usuarios:** `cuenta`, `nombre_usuario`, `contrasena` y `activo`.  
- **niveles_acceso:** `codigo_nivel_acceso`, `nombre_corto`, `nombre_largo` y `descripcion_nivel_acceso`.  
- **registros_actividad:** `nombre_usuario`, `fecha_hora`, `tabla_afectada`, `llave_afectada` y `tipo_cambio`.  

Es crucial notar que, en esta primera iteración, **`semestres`** *no aparece como una entidad independiente*. ¿Por qué? Porque en la visión cruda del cliente (el Insumo A y B), el "Semestre Año-1 (2026-1)" es percibido simplemente como una etiqueta de texto o un filtro temporal adjunto a la clase programada, no como un objeto de negocio con su propia identidad, ciclo de vida y atributos (fechas de inicio y fin). Esta "ceguera" es sumamente común en las primeras etapas del levantamiento de requisitos.

A partir de este punto y, como estándar de facto en el modelado de bases de datos, adoptamos la **notación *snake_case*** (minúsculas separadas por guiones bajos), en lugar de camelCase (primera letra mayúscula). En mi opinión, esta práctica responde a razones de compatibilidad nativa con motores de bases de datos (DBMS) y lenguajes de programación. Debe evitarse el uso de espacios, tildes, acentos, eñe o caracteres especiales que generan errores de codificación, es decir, deben cumplir el estandar SQL92. Ilustrativamente, en lugar de usar "Nombre Completo" o "NombreCompleto", utilizamos `nombre_completo`; en lugar de "Fecha de Nacimiento", usamos `fecha_nacimiento`. Esto garantiza un mapeo controlado y una precisión semántica cuando el concepto se origina en español, recordándonos que el lenguaje que usamos para modelar termina por estructurar el pensamiento del sistema. En la etapa final, haremos una transformación lingüística del español al inglés para garantizar un mapeo con precisión semántica universal.

En mi opinión, la revisión de esta primera iteración nos confronta invariablemente con lo que podríamos llamar las "trampas de la práctica cotidiana". Siguiendo los tips metodológicos que Captain nos advierte para no arrastrar inconsistencias a las fases subsecuentes, el diseñador debe mantenerse alerta ante tres síntomas clásicos de esta **"ceguera" inicial**: **primero**, los **atributos multivaluados o agrupados** (como `nombre_completo` o `direccion`), que no son más que una confusión donde múltiples propiedades se disfrazan de una sola, violando el principio de atomicidad; **segundo**, las **entidades ocultas** o no identificadas, aquellas que yacen subordinadas y confundidas con meros atributos de otras (como el "semestre" o el "periodo" incrustado como texto en la clase programada) y que requieren una abducción para emerger con identidad propia; y **tercero**, las **redundancias y dependencias** **funcionales cruzadas**, donde datos que pertenecen a un dominio (como el correo del profesor) son secuestrados por otro (la lista de grupo).

Dicho de otra manera, estos hallazgos no son meros errores técnicos, sino el reflejo de cómo la organización ha normalizado el desorden ordenado en sus hojas de cálculo. Para transitar hacia la segunda iteración, no basta con reorganizar; es imperativo aplicar el bisturí de la Primera Forma Normal (1NF de *1st Normal Form*) y descomponer estas estructuras, garantizando que cada atributo sea estrictamente atómico y que cada entidad sea dueña de su propia esencia.

##### 1.2 Técnica de Refinamiento

Para transitar de la visión cruda de la primera iteración a una estructura depurada en una segunda iteración, aplicamos un filtro lingüístico y lógico, valiéndonos de otro de los *tips* fundamentales sistematizados por Captain: la extracción y el cuestionamiento con la frase **"desear llevar el control de"**.

Primero, preguntamos: ¿De qué, específica e independientemente, debe llevarse un registro? Segundo, descomponemos los requisitos y las anomalías de los reportes en oraciones con una estructura **Sujeto - Verbo - Complemento (Objeto)**. Es aquí donde el **razonamiento abductivo** propuesto por Charles S. Peirce opera como una "***intuición racional***" para resolver los hechos sorprendentes (las anomalías de los reportes):

- El [departamento de registro académico] \<debe llevar el control de\> las [clases programadas], los [cursos] y [docentes]  
- Los [estudiantes] \<se inscriben en\> [clases programadas]  
- Los [cursos] \<se programan\> cada [semestre]  
- Los [docentes] \<son asignados\> a cada [clase programada]  
- La [aplicación] \<debe llevar el control\> de la [actividad de los usuarios]  

Análisis:

- *Ejemplo 1: Incorporación de una entidad oculta:*  
  Al analizar el Insumo A, notamos que las clases dependen de un periodo. Formulamos: "El [jefe de registro académico] (Sujeto) \<debe agrupar y controlar\> (Verbos) las [clases programadas] por [periodos académicos] o [**semestres**] (Complemento)".
  ¡Eureka! Mediante esta abducción pedagógica, **`semestres`** emerge como una entidad propia, con sus propias reglas y atributos. La anomalía se disuelve.

- *Ejemplo 2: Ruptura de la desnormalización:*  
  "El [*sistema*] (Sujeto) \<debe registrar\> (Verbo) la [*inscripción* de los *Estudiantes*] (Complemento 1) en las [*Clases Programadas*] (Complemento 2)".
  Esta descomposición revela que "Estudiantes" y "Clases Programadas" son entidades distintas. "lista_estudiantes" no es un atributo de la clase, sino el resultado de una relación que requiere sus propios datos (como la fecha de inscripción). Asimismo, el "Nombre del Profesor" en la lista de grupo debe romperse, pues el Docente es una entidad que "imparte" o "se asigna" a la clase, no un mero atributo de texto de la misma.

- Los [usuarios] \<deben tener\> diferentes [niveles de acceso]  

Esta técnica nos fuerza a cuestionar la agrupación de datos y nos prepara para la aplicación de reglas de integridad.

##### 1.3 El imperativo de la atomicidad: Hacia la Primera Forma Normal (1NF)

Hasta aquí, hemos capturado la visión del cliente. Sin embargo, desde una perspectiva técnica y metodológica, debemos cuestionar la estructura de estos atributos. ¿Qué datos necesitamos realmente y cómo deben estar organizados para que el sistema sea eficiente?

*La inclusión de un atributo no se decide por intuición, sino por necesidad operativa* y por una regla fundamental del diseño relacional: la **Primera Forma Normal (1NF)**. Esta regla establece que ***todos los atributos de una entidad deben ser atómicos***; es decir, *cada atributo debe describir una y solo una característica o propiedad de la entidad*, sin agrupar múltiples valores en una sola columna y cada entidad debe poseer una llave o *clave primaria* que identifique de manera única cada registro.

Al revisar la primera iteración, identificamos violaciones a este principio:  
1. `nombre_completo` agrupa el nombre o nombres y los apellidos paterno y materno. ¿Qué sucede si necesitamos buscar a todos los estudiantes con el apellido "García"? Con el campo agrupado, la búsqueda es ineficiente y propensa a errores.  
2. `direccion` agrupa la calle, la colonia, la ciudad, el estado, el país y el código postal. Esto impide realizar reportes segmentados por región o validar códigos postales de manera automatizada.  
3. La ausencia de un identificador único (`id_...`) impide distinguir entre dos entidades que puedan compartir el mismo nombre o código de negocio, violando la integridad de entidad.  

Por lo anterior, es plausible afirmar que resulta imperativo *descomponer estos atributos compuestos* en sus elementos atómicos constitutivos y asignar claves primarias antes de avanzar. El atributo `direccion` se debe descomponer en `calle`, `colonia`, `ciudad`, `estado`, `pais` y `codigo_postal`, pensando en que operativamente sea útil para segmentación geográfica y validación postal.

##### 1.4 Acotamiento conceptual por entidad (Iteración 2: La versión depurada)

Aplicando la regla de la 1NF y la técnica Sujeto-Verbo-Complemento, reestructuramos las entidades para garantizar la atomicidad, la identificación única y evitar solapamientos. A continuación, se presenta el desglose definitivo de atributos para cada entidad, utilizando la notación *snake_case*, en estricto cumplimiento de la **Primera Forma Normal** (**1NF**), la cual establece que todo atributo debe ser atómico (indivisible), describiendo una y solo una característica de la entidad sin agrupar múltiples valores en una misma columna y que cada tabla debe poseer una **clave primaria** que identifique de manera unívoca cada ocurrencia, eliminando así cualquier redundancia o grupo repetitivo.

**1. estudiantes**

Individuo matriculado que escoge la oferta académica. Sus atributos capturan identificación personal, contacto y ubicación de forma atómica.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_estudiante` | Clave primaria. Referencia única interna para localizar registros sin depender de datos variables. |
| `num_seguro_social` | Dato administrativo institucional para cumplimiento normativo. |
| `apellido_paterno` | Componente atómico esencial para ordenamiento y búsqueda formal. |
| `apellido_materno` | Componente atómico esencial para ordenamiento y búsqueda formal. |
| `nombre` | Primer nombre de pila. Se mantiene separado para respetar la atomicidad. |
| `segundo_nombre` | Campo opcional que permite flexibilidad cultural sin romper la estructura atómica. |
| `genero` | Clasificación demográfica para reportes institucionales y segmentación estadística. |
| `fecha_nacimiento` | Dato temporal base para validación de requisitos de admisión y cálculos etarios. |
| `correo_electronico` | Canal digital principal. Único por estudiante para notificaciones. |
| `telefono_celular` | Contacto prioritario para emergencias y comunicación móvil. |
| `telefono_casa` | Contacto residencial alternativo o de respaldo. |
| `telefono_trabajo` | Contacto laboral, útil para estudiantes que ejercen funciones administrativas o docentes. |
| `calle` | Calle y número. Requerido para correspondencia oficial. |
| `colonia` | Colonia que evita columnas multivaluadas. |
| `ciudad` | Componente geográfico para segmentación regional. |
| `estado` | Complemento territorial para clasificación regulatoria. |
| `pais` | País para distinguir entre estudiantes nacionales y extranjeros en el futuro. |
| `codigo_postal` | Optimiza la validación y clasificación de correspondencia física. |

**2. cursos**

Unidad académica curricular independiente del tiempo y el espacio. Define identidad, denominación y contenido pedagógico.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_curso` | Clave primaria. Referencia interna para relacionar instancias sin depender de códigos cambiantes. |
| `codigo_curso` | Identificador alfanumérico institucional (ej. `mat101`). Sirve para catalogación. |
| `nombre_corto` | Denominación abreviada para listados y menús compactos. |
| `nombre_largo` | Denominación oficial completa para catálogos formales. |
| `descripcion_curso` | Texto extenso con objetivos y temario. Atributo de contenido, no de identificación. |

**3. semestres**

*Entidad emergente mediante la abducción pedagógica.* Periodo académico calendario que estructura la oferta educativa. Define identidad temporal y límites cronológicos.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_semestre` | Clave primaria. Referencia interna para auditoría y relaciones temporales. |
| `ciclo_academico` | Orden ordinal dentro del plan de estudios (año de inicio-año de término, ej. 2025-2026). |
| `semestre` | Denominación institucional específica, ej. "1" o "2". |
| `fecha_inicio` | Límite inferior que marca el arranque de inscripciones y actividades. |
| `fecha_fin` | Límite superior que cierra evaluaciones, periodos y reportes. |

**4. clases_programadas**

Instancia temporal y espacial concreta de un curso dentro de un semestre. Captura variables logísticas de ejecución.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_clase` | Clave primaria. Referencia única para vincular inscripciones y asignaciones de docentes. |
| `codigo_horario` | Etiqueta legible para consulta humana (ej. `mat101-lun-08`). |
| `seccion` | Subgrupo dentro de un mismo curso y horario. Permite ofertas paralelas. |
| `dia` | Variable atómica que especifica el día o días de la semana de impartición. |
| `hora` | Bloque horario de inicio/fin. Separado de `dia` para cumplir estrictamente la atomicidad. |
| `ubicacion` | Aula, laboratorio o enlace virtual. Contexto de ejecución. |

**5. docentes**

Personal académico responsable de la impartición. Se delimita por su rol profesional y contractual.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_docente` | Clave primaria. Referencia única para gestión de cargas académicas y evaluaciones. |
| `num_empleado` | Número de empleado interno. |
| `num_seguro_social` | Número de seguro social para fines contractuales o de nómina. |
| `apellido_paterno` | Componente atómico del nombre personal. |
| `apellido_materno` | Componente atómico del nombre personal. |
| `nombre` | Componente atómico del nombre personal. |
| `segundo_nombre` | Componente opcional y atómico. |
| `genero` | Clasificación demográfica para reporting institucional. |
| `correo_electronico` | Canal principal de comunicación académica y coordinación. |
| `telefono_celular` | Contacto directo para coordinación o emergencias. |
| `telefono_casa` | Contacto residencial alternativo. |
| `telefono_trabajo` | Contacto en oficina o departamento académico. |
| `acerca_de` | Campo libre para perfil profesional o biografía académica. |

**6. usuarios**

Entidad de seguridad y autenticación. Gestiona credenciales, permisos y estado de la cuenta. Se separa de los datos personales para cumplir con el principio de mínima exposición.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_usuario` | Clave primaria. Referencia interna que vincula la cuenta de seguridad con los perfiles de negocio. |
| `cuenta` | Identificador único de autenticación (login). |
| `nombre_usuario` | Denominación visible en interfaz o reportes de auditoría. |
| `contrasena` | Credencial cifrada para validación de identidad. Nunca visible en texto plano. |
| `activo` | Indicador de estado operativo (1=Activo, 0=Inactivo). Permite deshabilitar acceso sin borrar historial. |

**7. niveles_acceso**

Matriz de permisos que define jerarquías o roles de seguridad sin contener lógica de aplicación ni datos personales.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_nivel_acceso` | Clave primaria. Referencia interna para garantizar integridad referencial con `usuarios`. |
| `codigo_nivel_acceso` | Etiqueta institucional del rol (ej. `admin`, `estudiante`, `docente`). |
| `nombre_corto` | Denominación abreviada para interfaces o consultas rápidas. |
| `nombre_largo` | Denominación completa para documentación y políticas institucionales. |
| `descripcion_nivel_acceso` | Detalle de permisos, alcances y responsabilidades asociadas al rol. |

**8. registros_actividad**

Registro histórico de eventos de autenticación y cambios en el sistema (auditoría). Solo captura el cuándo y el qué del evento.

| Atributo (*snake_case*) | Justificación conceptual |
| :--- | :--- |
| `id_registro_actividad` | Clave primaria. Referencia única secuencial para trazabilidad. |
| `fecha_hora_inicio` | Marca temporal del evento de autenticación exitosa o del cambio. |
| `fecha_hora_fin` | Marca temporal de cierre de sesión o expiración. Permite calcular duración. |
| `tabla` | Nombre de la tabla de la base de datos que fue afectada por la acción (ej. `estudiantes`, `cursos`). |
| `pk_tabla` | Valor de la clave primaria del registro específico que fue modificado, permitiendo rastrear el cambio exacto. |
| `cambio` | Tipo de operación realizada: `insercion`, `actualizacion` o `borrado`. |
| `detalles` | Detalles del cambio. |

##### 1.5 Selección de identificadores y claves primarias

Una vez definido qué vamos a rastrear y cómo lo describiremos de manera atómica, necesitamos un mecanismo confiable para señalar cada registro individualmente. Aquí es donde seleccionamos la **llave** o **clave primaria** (PK, de *Primary Key*).

Siguiendo la recomendación del autor, priorizamos **identificadores numéricos** secuenciales autoincrementales (llamadas *surrogate keys*) en lugar de códigos alfanuméricos del negocio. ¿Por qué? Porque los índices sobre números enteros son computacionalmente más eficientes, garantizan unicidad absoluta y no dependen de reglas institucionales que podrían cambiar (como un código de curso que se reestructura o un número de seguro social que se migra). Es, en esencia, una decisión pragmática frente a la volatilidad del mundo real y la construcción social de los códigos institucionales.

**Claves primarias asignadas (en *snake_case*):**
- `id_estudiante` para `estudiantes`
- `id_curso` para `cursos`
- `id_semestre` para `semestres`
- `id_clase` para `clases_programadas`
- `id_docente` para `docentes`
- `id_usuario` para `usuarios`
- `id_nivel_acceso` para `niveles_acceso`
- `id_registro_actividad` para `registros_actividad`

**Regla de validación final del Paso 1:** Antes de cerrar el Paso 1, aplicamos la prueba de fuego de la normalización: «¿Cada atributo depende exclusivamente de la entidad y no de otra?». Si algún atributo responde con un "no", no pertenece a esa entidad y debe reubicarse. Esta verificación asegura que el modelo conceptual esté listo para derivar relaciones en el Paso 2 sin arrastrar dependencias cruzadas o redundancias.

Con las entidades descubiertas (incluyendo la emergencia de `semestres`), sus atributos acotados atómicamente y sus identificadores definidos bajo un estándar de nomenclatura consistente, el Paso 1 está completo. El modelo está listo para pasar al Paso 2, donde conectaremos estos objetos mediante relaciones unarias y binarias, transitando de la estática de las entidades a la dinámica de los vínculos organizacionales.

---

#### Paso 2: Identificación de relaciones unarias y binarias

En mi opinión, el gran error de los diseñadores novatos es creer que las entidades viven en el vacío conceptual. Una vez que hemos descubierto los objetos de interés y sus atributos atómicos en el Paso 1, nos enfrentamos a una realidad ineludible: en el tejido social y operativo de cualquier organización, **las entidades no existen aisladas; su verdadero valor y significado emergen de los vínculos que las conectan**. Como bien apunta Fidel A. Captain: «Una relación es lo que existe entre entidades o entre Relaciones (tablas). Se utilizan verbos para describir relaciones, las cuales pueden ser de uno a uno, uno a muchos o muchos a muchos» (Captain, 2015, p. 38).

Dicho de otra manera, si el Paso 1 fue un ejercicio de *identidad del dominio* (identificar el "ser" de los datos), el Paso 2 es un ejercicio de *praxeología y dinámica organizacional* (comprender el "hacer" y el "interactuar"). La separación arquitectónica que establecimos previamente entre la capa operativa de seguridad (*backend*) y la capa académica de negocio (*frontend*) nos servirá ahora como un filtro conceptual muy útil. Esta distinción no solo ordena el descubrimiento de relaciones, sino que nos protege de caer en la trampa de crear vínculos indirectos que, más adelante, contaminarían la implementación física con redundancias estructurales, creando un auténtico monstruo de Frankenstein. La herramienta que nos guiará en esta tarea es la **Matriz Entidad-Entidad (Matriz E-E)**.

##### Tipos de relaciones

Antes de armar la matriz, conviene tener claros los dos tipos de relaciones que Captain trabaja en esta etapa. Identificarlos correctamente desde el inicio determinará en gran medida la solidez del modelo final:

- **Relaciones binarias:** Son aquellas en las que intervienen dos entidades distintas. Por ejemplo, los `docentes` se asignan a `clases_programadas`. Son, con mucho, las más comunes y representan contratos de negocio entre áreas funcionales diferentes.
- **Relaciones unarias (o recursivas):** Ocurren cuando una entidad se vincula consigo misma. Por ejemplo, `cursos` puede relacionarse consigo misma a través de la relación de prerrequisito: un curso es prerrequisito de otro curso. Aunque al principio pueden parecer contraintuitivas, resultan indispensables para modelar jerarquías, dependencias o recursividades dentro de un mismo concepto, casi como un sistema social observándose a sí mismo.

**Nota:** Captain aclara que «la teoría de bases de datos permite relaciones de orden superior, entre tres o más entidades. Sin embargo, este libro ignora este tipo de relaciones porque las relaciones unarias y binarias suelen ser suficientes para responder cualquier consulta que se realice sobre los datos en la base de datos» (2015, p. 38). En mi opinión, esta es una decisión pragmática brillante: simplifica el modelado sin sacrificar la capacidad expresiva, evitando el reduccionismo de querer abarcarlo todo en una sola estructura compleja e inmanejable.

##### Paso 2-1: Construir la Matriz Entidad-Entidad

La Matriz E-E es, en esencia, una herramienta de cruce bidireccional que nos asegura no dejar ningún par de entidades sin evaluar. Se trata de una tabla con el mismo número de filas y columnas, donde cada entidad descubierta en el Paso 1 encabeza tanto una fila como una columna. Cada intersección representa una relación potencial.

Para mantener la coherencia con la arquitectura separada entre dominios, agrupamos visualmente las entidades por dominio antes de trazar la matriz, reflejando la dualidad del sistema sociotécnico:

| Dominio Operativo (Backend / Seguridad) | Dominio Académico (Frontend / Negocio) |
| :--- | :--- |
| `usuarios` | `estudiantes` |
| `niveles_acceso` | `docentes` |
| `registros_actividad` | `cursos` |
| | `clases_programadas` |
| | `semestres` |

> **Regla de oro:** *Verifica que cada entidad descubierta esté listada en el encabezado de fila y encabezado de columna y que el orden de las entidades sea el mismo.* (Captain, 2015, p. 40)

##### Paso 2-2: Completar la Matriz e Identificar Vínculos

Cada celda de la Matriz E-E representa una relación potencial. Para descubrir qué relaciones existen realmente, hay que volver al dominio del problema y cuestionar minuciosamente al cliente. En la práctica, esto implica recorrer la matriz celda por celda, aplicando la técnica del **Sujeto-Verbo-Complemento** que afinamos en el Paso 1.

**La Regla de Lectura (De Fila a Columna)**

Captain establece una regla estricta para evitar ambigüedades semánticas. La pregunta rectora para cada celda es: «¿Está [Entidad en Encabezado de Fila] relacionada con [Entidad en Encabezado de Columna]?». Si existe una relación, se coloca un verbo en la celda.

**Notación**

En congruencia con nuestra decisión metodológica del Paso 1, **el modelo conceptual (Pasos 1 al 5) se trabaja íntegramente en español y en notación *snake_case***. Esto evita la prematurez técnica y mantiene la fidelidad semántica con el *orgware* del cliente. La transformación lingüística al inglés, necesaria para la compatibilidad nativa con los RDBMS, se reserva como un cambio de paradigma deliberado para el **Paso 6** (implementación física).

Por lo tanto, en esta etapa trabajaremos con dos niveles de abstracción en español:

1.  **Validación Semántica (Lenguaje Natural):** Verbos conjugados en tercera persona para formar oraciones completas al leerse de *Fila → Columna*. Su propósito es que el cliente comprenda y valide la regla de negocio de inmediato.
2.  **Abstracción Técnica (Diccionario Conceptual):** Verbos en infinitivo estandarizado (ej. `impartir`, `inscribir`, `ser_prerrequisito`). Su propósito es definir el nombre de la relación para el modelo conceptual, manteniendo la precisión semántica sin romper la convención del idioma del dominio del problema.

**Simetría y Poda de la Matriz**

Inicialmente, al revisar cada celda, las relaciones se duplican en la mitad superior e inferior de la Matriz (imagen especular). Captain instruye: «Ignora la mitad superior de la Matriz trazada a lo largo de la diagonal... ya que es una imagen especular de la mitad inferior» (2015, p. 41). Deberá examinarse cada relación detenidamente para asegurar que es una relación que se desea capturar y que no se trata de una redundancia.

A continuación, se presenta la Matriz E-E resultante para el sistema académico, depurada y lista para la validación con el cliente:

**Matriz Entidad-Entidad (Validación Semántica y Abstracción Técnica)** *(Lectura: [Entidad en Fila] + Verbo Conjugado + [Entidad en Columna])*

| | `estudiantes` | `cursos` | `clases_programadas` | `semestres` | `docentes` | `usuarios` | `niveles_acceso` | `registros_actividad` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`estudiantes`** | — | | se inscriben en<br>(0:N / 0:N)<br>*(inscribir)* | | | son autenticados por<br>(1:1 / 0:1)<br>*(autenticar)* | | |
| **`cursos`** | | \* son prerrequisito de<br>(0:N / 0:N)<br>*(ser_prerrequisito)* | son materializados en<br>(0:N / 1:1)<br>*(materializar)* | | son impartidos por<br>(0:N / 0:N)<br>*(impartir)* | | | |
| **`clases_programadas`** | | | — | pertenecen a<br>(1:1 / 0:N)<br>*(pertenecer)* | se asignan a<br>(1:1 / 0:N)<br>*(asignar)* | | | |
| **`semestres`** | | | | — | | | | |
| **`docentes`** | | | | | — | son autenticados por<br>(1:1 / 0:1)<br>*(autenticar)* | | |
| **`usuarios`** | | | | | | \* crean<br>(0:N / 1:1)<br>*(crear)* | autorizan a<br>(1:1 / 0:N)<br>*(autorizar)* | generan<br>(0:N / 1:1)<br>*(generar)* |
| **`niveles_acceso`** | | | | | | | — | |
| **`registros_actividad`** | | | | | | | | — |

**Nota:** Las celdas en blanco en la mitad superior se mantienen intencionalmente vacías siguiendo la regla de Captain. Las celdas con `—` en la diagonal principal también se omiten por convención, salvo las relaciones unarias que se leerán en su fila/columna correspondiente.

##### Análisis de Matices: La "Ceguera" Relacional y la Abducción

Al observar la matriz resultante, es natural que surjan dudas o que el cliente señale aparentes anomalías. Una mirada cercana a la especificación del problema y a la lógica organizacional responde a estas aparentes anomalías, validando la correcta aplicación de la metodología. Es aquí donde el razonamiento abductivo de Peirce opera como una "intuición racional" para resolver los "hechos sorprendentes":

1.  **¿Por qué los `estudiantes` no `se inscriben en` los `cursos` directamente?**
    Este es un error clásico de modelado, una confusión conceptual. Los `cursos` son entes abstractos (el catálogo de "Cálculo I"); los estudiantes no se inscriben en el concepto, sino en la *instancia temporal y espacial* de ese curso. La relación válida es: Estudiantes se inscriben en Clases Programadas. Capturar un vínculo directo entre estudiantes y cursos duplicaría lógica y rompería la integridad del proceso de programación académica.

2.  **¿Por qué los `docentes` se `asignan` a `clases_programadas` e `imparten` `cursos`?**
    Aquí aplicamos la regla del "mismo verbo, diferente contexto". Los docentes imparten el contenido curricular a nivel académico (`cursos`), pero se asignan a las sesiones logísticas según lo establecido en el horario (`clases_programadas`). Son dos contratos de negocio distintos que deben capturarse por separado.

3.  **¿Por qué los `niveles_acceso` y los `registros_actividad` solo se relacionan con los `usuarios`?**
    Porque los permisos y la auditoría operan exclusivamente sobre la *identidad digital* (la cuenta de seguridad), no sobre el perfil biográfico o académico (Estudiante/Docente). Habrá una relación *indirecta* entre `niveles_acceso` y `estudiantes` a través de `usuarios`, pero capturar un vínculo directo violaría la separación de responsabilidades entre el *frontend* y el *backend*, creando un modelo frágil ante los cambios organizacionales.

Estos son matices que solo pueden verificarse examinando la Matriz detenidamente. La importancia de eliminar las relaciones redundantes no puede ser subestimada.

> **Recuerda:** Verifica con el cliente que cada relación representada en la Matriz E-E sea válida. No crees relaciones que no sean necesarias ni estén capturadas en el dominio del problema. Estás diseñando la base de datos para tu cliente y no para ti mismo, por lo que deseas capturar las relaciones tal como las ve el cliente y no como las ves tú. (Captain, 2015, p. 47)

##### Clasificación Funcional por Dominios de Interacción

La separación arquitectónica entre la capa académica y la capa operativa permite clasificar los vínculos registrados en tres grupos funcionales, facilitando la validación con el cliente y preparando el terreno para una implementación modular:

1.  **Relaciones Intra-Frontend (Lógica Académica)**
    Modelan exclusivamente los procesos curriculares y de inscripción:
    - `estudiantes` ↔ `clases_programadas` (`inscribir`): La inscripción ocurre sobre la instancia programada.
    - `clases_programadas` ↔ `cursos` (`materializar`): Una clase programada es la materialización temporal y espacial de un curso.
    - `clases_programadas` ↔ `semestres` (`pertenecer`): El horario pertenece a un periodo académico específico.
    - `docentes` ↔ `cursos` (`impartir`): El docente imparte el contenido curricular a nivel académico.
    - `cursos` ↔ `cursos` (`ser_prerrequisito`): Relación unaria que captura la dependencia académica.

2.  **Relaciones Intra-Backend (Capa de Seguridad y Auditoría)**
    Operan exclusivamente sobre la gestión de identidades y el control de acceso:
    - `usuarios` ↔ `usuarios` (`crear`): Relación unaria que refleja la jerarquía administrativa (una cuenta autorizada crea nuevas cuentas).
    - `niveles_acceso` ↔ `usuarios` (`autorizar`): Los permisos se asignan a las cuentas de acceso, preservando el principio de mínima exposición.
    - `registros_actividad` ↔ `usuarios` (`generar`): La auditoría registra eventos de autenticación asociados a una cuenta, no a un individuo físico.

3.  **Relaciones Puente (Conexión Frontend ↔ Backend)**
    Vínculos necesarios para traducir la identidad operativa en perfiles de negocio:
    - `usuarios` ↔ `estudiantes` (`autenticar`): Vincula la cuenta de seguridad con el perfil académico del estudiante.
    - `usuarios` ↔ `docentes` (`autenticar`): Vincula la cuenta de seguridad con el perfil académico del docente.
    - `clases_programadas` ↔ `docentes` (`asignar`): Cierra el ciclo entre la lógica curricular y la ejecución logística.

##### Depuración y Validación Crítica

Un error frecuente en etapas tempranas es registrar relaciones indirectas o transversales que no existen en el dominio real. La matriz nos permite detectarlas y eliminarlas antes de que contaminen el modelo:

- **`niveles_acceso` ↔ `estudiantes` / `docentes`**: No existe relación directa. Los permisos operan sobre la capa de autenticación (`usuarios`). Asignarlos directamente a perfiles personales violaría la separación de responsabilidades y complicaría la gestión de roles (ej. ¿qué pasa si un estudiante se convierte en docente? Si el nivel de acceso está atado al perfil personal, habría que migrar datos; si está atado al `usuario`, solo se actualiza un registro).
- **`registros_actividad` ↔ `clases_programadas`**: No existe relación directa. La auditoría rastrea accesos al sistema y cambios en la base de datos (mediante los atributos `tabla` y `pk_tabla` que definimos en el Paso 1), no interacciones de negocio con entidades académicas específicas.

Esta depuración, validada iterativa y constantemente con el cliente, garantiza que el modelo capture únicamente las interacciones esenciales, alineándose con la premisa de Captain sobre capturar la visión del cliente.

Con la Matriz Entidad-Entidad completada bajo la estricta regla de lectura de fila a columna, validada en lenguaje natural con el cliente y estructurada bajo una arquitectura de dominios separados, hemos mapeado exhaustivamente cómo se comunican los objetos del dominio. Cada verbo registrado en la matriz representa un contrato de negocio verificable y una futura restricción de integridad referencial.

Este mapa de interacciones está listo para traducirse en el **Paso 3: Crear un diagrama Entidad-Relación (E-R) simplificado**, donde construiremos una representación visual que sintetizará estas conexiones de manera intuitiva. En esa etapa, introduciremos la notación gráfica (rectángulos para entidades y rombos para relaciones) sin incorporar aún la opcionalidad ni la cardinalidad, dejando ese refinamiento cuantitativo para los pasos subsiguientes. Transitaremos, así, de la lógica textual a la topología visual.

---

#### Fase II. Modelado conceptual (Pasos 3, 4 y 5)

Habiendo cartografiado el territorio, procedemos a formalizar la visión del usuario. Esta fase traduce los hallazgos previos en una arquitectura visual (el diagrama E-R), inyectando progresivamente las reglas de negocio —obligatoriedad y cardinalidad— hasta consolidar un modelo conceptual maduro, validado y blindado contra las ambigüedades del lenguaje natural.

#### Paso 3: Diagrama Entidad-Relación simplificado

Una vez que hemos descubierto los objetos de interés, acotado sus propiedades atómicas y mapeado exhaustivamente cómo interactúan entre sí mediante la Matriz Entidad-Entidad, es momento de dar forma visual a esta estructura conceptual. El Paso 3 tiene un propósito metodológico claro y delimitado en la técnica de Captain: trasladar la información derivada de los pasos anteriores a un diagrama gráfico inicial, liberado deliberadamente de restricciones de opcionalidad y cardinalidad.

Según Fidel A. Captain (2015, p. 52), este diagrama simplificado constituye la primera manifestación tangible del modelo conceptual; captura la visión del usuario sin saturarla con detalles de implementación que abordaremos en etapas posteriores: «Los modelos conceptuales se ocupan de la naturaleza lógica de los datos y de qué se está representando». En mi opinión, este paso materializa exactamente esa naturaleza lógica, actuando como un puente conceptual entre la narrativa social del cliente y la arquitectura formal del sistema, priorizando la claridad estructural sobre el detalle operativo.

##### 3.1 Notación

En la metodología de los Seis Pasos, cada paso tiene una responsabilidad específica. El diagrama Entidad-Relación (E-R) simplificado, por diseño pedagógico y arquitectónico, **no contiene información sobre obligatoriedad ni cardinalidad**. Esta exclusión no es una omisión técnica, sino una decisión deliberada: al posponer las reglas de negocio cuantitativas, evitamos la sobrecarga cognitiva, mejoramos la abstracción y nos aseguramos de que la estructura lógica subyacente sea sólida antes de inyectar restricciones. Dicho de otra manera, primero debemos cartografiar el territorio antes de dibujar sus fronteras.

Es imperativo introducir desde esta etapa el estándar visual que regirá la transición hacia el Modelo Relacional (Paso 6): la **Notación Pata de Gallo** (*Crow's Foot Notation*). Desarrollada originalmente por Gordon Everest (1976) y ampliamente adoptada en la industria, esta notación utiliza símbolos intuitivos en los extremos de las líneas de relación para representar gráficamente la cardinalidad y la obligatoriedad entre entidades, símbolos que no usaremos en este momento, ya que es una versión simplificada. Nos limitamos a usar una línea simple que une las entidades, para evitar la saturación visual, con el verbo sobre la línea. Tener presente esta notación nos prepara metodológicamente para el Paso 5 (el Diagrama Entidad-Relación Detallado) y el Paso 6 (Modelo Relacional), donde todos los símbolos de la notación cobrarán vida para cimentar la integridad referencial de los datos.

> *Nota:* «No necesitas incluir todos los atributos para todas las entidades en el diagrama E-R simplificado porque hacerlo puede saturar el diagrama haciendo que las entidades sean inusualmente largas. Por lo tanto, en interés del espacio, *solo la clave primaria y los atributos más importantes y relevantes* deben incluirse en las entidades del diagrama E-R simplificado» (Captain, 2015, p. 52).

Para construir el diagrama de manera sistemática y verificable, seguimos dos acciones secuenciales tal como las prescribe el autor:

1.  **Crear las entidades (rectángulos):** Cada una de las entidades descubiertas en el Paso 1 se representa mediante un rectángulo, indicando claramente la clave primaria y otros atributos importantes. Captain recomienda usar sustantivos en plural para los nombres de las entidades, práctica que mantenemos por convención y que facilita la lectura de las relaciones en etapas posteriores.
2.  **Crear las relaciones con una línea que une las entidades:** Cada verbo derivado en el Paso 2 (Matriz E-E) se representa mediante un diamante con su nombre en el interior. Las entidades se conectan a las relaciones según lo validado en la matriz, reorganizando la disposición espacial hasta que el flujo de conexiones sea intuitivo y libre de solapamientos innecesarios.

> **Recuerda:** Asegúrese de que cada entidad tenga una clave primaria y que los atributos importantes estén representados en esa entidad. Además, asegúrese de que cada relación tenga un nombre, que el nombre sea correcto y que esté asociada con las entidades correctas. (Captain, 2015, p. 54).

##### 3.2 Entidades y atributos estratégicos

A continuación, presentamos las entidades con sus claves primarias y los atributos estratégicos que deben aparecer en el diagrama, integrando nuestra separación de dominios *frontend* (académico) y *backend* (operativo). Los atributos marcados en **negrita** son candidatos a índices secundarios o claves de búsqueda frecuente.

**Dominio Académico (Frontend)**

| Entidad | Clave Primaria | Atributos estratégicos (según Captain) |
| :--- | :--- | :--- |
| `estudiantes` | `id_estudiante` | `correo_electronico`, **`apellido_paterno`**, **`nombre`**, `fecha_nacimiento` |
| `cursos` | `id_curso` | `codigo_curso`, `nombre_corto`, `nombre_largo` |
| `clases_programadas` | `id_clase` | `codigo_horario`, `seccion`, `dia`, `hora`, `ubicacion` |
| `semestres` | `id_semestre` | `ciclo_academico`, `semestre`, `fecha_inicio`, `fecha_fin` |
| `docentes` | `id_docente` | `correo_electronico`, **`apellido_paterno`**, **`nombre`**, `acerca_de` |

> ***Nota:*** Aunque Captain mantiene en su obra original `CourseCode` como clave primaria para `Courses`, en nuestra modelación priorizamos `id_curso` (clave subrogada numérica) siguiendo la regla establecida en el Paso 1. Esta decisión pragmática responde a la eficiencia computacional de los índices sobre enteros y nos protege de la volatilidad de los códigos institucionales, un riesgo que el propio autor reconoce.

**Dominio Operativo (Backend)**

| Entidad | Clave Primaria | Atributos estratégicos (según Captain) |
| :--- | :--- | :--- |
| `usuarios` | `id_usuario` | **`cuenta`**, **`nombre_usuario`**, `contrasena`, `activo` |
| `niveles_acceso` | `id_nivel_acceso` | **`codigo_nivel_acceso`**, `nombre_corto`, `nombre_largo` |
| `registros_actividad` | `id_registro_actividad` | `fecha_hora_inicio`, `fecha_hora_fin` |

##### 3.3 Relaciones derivadas y topología del diagrama

Las relaciones que conectan estas entidades provienen directamente de la Matriz E-E completada en el Paso 2. Captain enfatiza que *cada verbo registrado representa un contrato de negocio verificable*. La separación arquitectónica nos permite clasificarlas en tres grupos funcionales:

- **Relaciones Intra-Frontend (Lógica Académica):** `estudiantes` ↔ `clases_programadas` (`inscribir`), `clases_programadas` ↔ `cursos` (`materializar`), `clases_programadas` ↔ `semestres` (`pertenecer`), `docentes` ↔ `cursos` (`impartir`), `cursos` ↔ `cursos` (`ser_prerrequisito`).
- **Relaciones Intra-Backend (Capa de Seguridad y Auditoría):** `usuarios` ↔ `usuarios` (`crear`), `niveles_acceso` ↔ `usuarios` (`autorizar`), `registros_actividad` ↔ `usuarios` (`generar`).
- **Relaciones Puente (Frontend ↔ Backend):** `usuarios` ↔ `estudiantes` (`autenticar`), `usuarios` ↔ `docentes` (`autenticar`), `clases_programadas` ↔ `docentes` (`asignar`).

##### 3.4 Topología del Diagrama E-R simplificado

Captain señala que las entidades con relaciones consigo mismas pueden representarse en el diagrama más de una vez para evitar cruces confusos o simplemente haciendo que la línea se conecte con la misma entidad. A continuación, presentamos la topología del diagrama para garantizar la máxima claridad conceptual:

```text
[niveles_acceso] ──<autorizan a>── [usuarios] ──<crean>── [usuarios]*
                         │
                         ├──<autentican a>── [estudiantes]
                         ├──<autentican a>── [docentes]
                         └─<generan>── [registros_actividad]

[semestres] ──<pertenecen a>── [clases_programadas] ──<se asignan a>── [docentes]
                                      │
                                      ├──<materializan>── [cursos] ──<son prerrequisito de>── [cursos]*
                                      │         │
                                      │         └──<son impartidos por>── [docentes]
                                      └─<se inscriben a>── [estudiantes]
```

> **Nota:** Los corchetes `[ ]` representan entidades; los ángulos `< >` representan relaciones. La notación `*` indica duplicación visual para clarificar relaciones unarias, priorizando la legibilidad sobre el formalismo estricto.

#### 3.5 Validación

Antes de considerar este paso concluido, Captain recomienda una verificación cruzada rápida pero indispensable (2015, p. 54). Actúa como nuestro filtro metodológico:

- ✅ ¿Cada entidad tiene una clave primaria claramente identificada y destacada?
- ✅ ¿Los atributos representados son realmente los que el cliente necesita consultar o filtrar con frecuencia?
- ✅ ¿Cada relación tiene un verbo preciso y está conectada exclusivamente a las entidades que participaron en la Matriz E-E?
- ✅ ¿Las relaciones unarias se representan sin generar ambigüedad visual (mediante duplicación de entidad o ciclo claro)?
- ✅ ¿El diagrama puede leerse de izquierda a derecha o de arriba hacia abajo sin requerir saltos cognitivos excesivos?
- ✅ ¿Se ha evitado saturar las entidades con atributos secundarios que pertenecen al diccionario de datos, no al diagrama conceptual?

Si todas las respuestas son afirmativas, el modelo conceptual está listo para evolucionar. En el siguiente paso, inyectaremos las reglas de negocio mediante **obligatoriedad y cardinalidad**, transformando este esquema estático en un modelo dinámico, validado y preparado para su posterior traducción al lenguaje de implementación.

#### 3.6 Consideraciones finales del Paso 3

La propuesta de Captain para este estudio de caso destaca por su equilibrio entre rigor metodológico y pragmatismo pedagógico. Tres elementos merecen especial atención en nuestra integración:

1.  **Selección estratégica de atributos:** Captain no incluye todos los atributos descubiertos en el Paso 1, sino **solo aquellos críticos** para la comprensión conceptual. Esto evita la saturación visual y mantiene el foco en la estructura lógica, recordándonos que un modelo no es un inventario, sino un mapa.
2.  **Manejo de relaciones unarias:** Para `es prerrequisito de` en `cursos` y `crean` en `usuarios`, la duplicación visual de la entidad o el ciclo claro priorizan la legibilidad. Es una concesión técnica al funcionamiento cognitivo del diseñador y del cliente.
3.  **Separación explícita de dominios:** Aunque Captain no explicita la separación *frontend/backend* en su notación original, la topología natural de sus diagramas agrupa las entidades operativas en un cluster distinto al académico. Hacerlo explícito refuerza la arquitectura del sistema y prepara el terreno para una implementación modular y segura.

Con esta validación, el diagrama Entidad-Relación simplificado queda listo para avanzar al Paso 4: Enumeración de afirmaciones, donde derivaremos la obligatoriedad y cardinalidad que transformarán esta estructura estática en un modelo dinámico de negocio. Transitaremos, así, de la topología visual a la semántica operativa.

---

#### Paso 4: Enumeración de afirmaciones

Una vez que hemos construido el diagrama Entidad-Relación simplificado en el Paso 3, contamos con una representación visual clara de las entidades y sus vínculos. Sin embargo, este diagrama aún guarda silencio sobre las reglas de negocio que gobiernan esas relaciones: ¿es imperativo que un estudiante tenga una cuenta de usuario? ¿Puede un curso existir sin prerrequisitos? ¿Cuántas clases puede impartir un docente como máximo o como mínimo?

El Paso 4 responde a estas interrogantes al inyectar semántica operativa al modelo, transformando vínculos abstractos en contratos de negocio verificables. Como señala Fidel A. Captain (2015, p. 66): «Las afirmaciones son necesarias para precisar los detalles sobre las relaciones entre las entidades en la base de datos. Las relaciones entre las entidades deben satisfacer las afirmaciones hechas sobre ellas. Además, el cliente debe verificar que cada afirmación hecha sea verdadera y correcta dentro del contexto de la base de datos que se está modelando».

En mi opinión, este paso representa el tránsito crucial de la estructura estática a la praxeología dinámica: ya no nos preguntamos solo "qué es" el dato, sino "cómo debe comportarse" el sistema ante la realidad organizacional. Es el momento en que la abstracción conceptual se somete al crisol de la lógica empresarial.

##### 4.1 Fundamentos conceptuales

Para derivar afirmaciones con rigor metodológico, es indispensable dominar los pilares conceptuales que Captain establece como invariables en su metodología. Ignorarlos no es un simple descuido técnico, sino una falla conceptual que arrastrará inconsistencias hasta la implementación física.

**4.1.1 La afirmación (*assertion*)**

Una afirmación se basa en el predicado matemático (una declaración de verdadero o falso). En el contexto de esta obra, es un predicado que siempre es verdadero. Por tanto, una afirmación se refiere a una declaración fáctica y verificable sobre dos (y solo dos) entidades en la base de datos que estamos intentando modelar, y la relación que existe entre ellas.

**4.1.2 Clase de entidad vs. Ocurrencia de entidad**

- **Clase de entidad:** Se refiere a las ocurrencias colectivas de todas las instancias o registros de una entidad, o a la entidad como concepto general (ej. `estudiantes`).
- **Ocurrencia de entidad:** Es una instancia real, tangible y específica de esa entidad (ej. "Ana García López" es una ocurrencia de `estudiantes`).

**4.1.3 Obligatoriedad y su reflejo físico (NULL / NOT NULL)**

La Obligatoriedad (denominada *Opcionalidad* en la obra de Captain) indica **qué puede y qué debe suceder en una relación**. Puede tomar uno de dos valores:

- `1` (**Obligatorio / Debe**): La relación es imperativa. No puede existir la ocurrencia de la entidad sin que este vínculo se cumpla. En la base de datos física, esto se traduce en que el campo de la clave foránea será estrictamente `NOT NULL`.
- `0` (**No obligatorio / Puede**): La relación no es un requisito. La ocurrencia de la entidad puede existir sin que este vínculo se establezca. En la base de datos física, esto permite que el campo de la clave foránea acepte un valor `NULL` (desconocido o no aplicable).

  *El matiz oculto del cero:* Captain advierte que tener una obligatoriedad de `0` (puede) infiere la posibilidad real de que algo no pueda suceder en una relación. *Siempre verifique con el cliente qué puede y qué no puede suceder.*

**4.1.4 Cardinalidad**

La cardinalidad indica el número de ocurrencias de entidad que pueden participar en una relación. En esta metodología, se restringe a dos valores:

- `1` (Solo uno): La relación se limita a una única ocurrencia.
- `N` (Muchos o al menos uno): La relación puede involucrar múltiples ocurrencias.

**4.1.5 Consejos prácticos para la redacción ("Captain-isms")**

Al revisar minuciosamente el Capítulo 5 de la obra original, existen tres detalles de sintaxis y redacción cruciales para el estudio de caso. Integrarlos le dará a nuestra narrativa un toque de autenticidad y rigor práctico, blindando el modelo contra la ambigüedad semántica:

1.  **Desambiguación de roles mediante corchetes `[ ]`:** Cuando una entidad participa en múltiples relaciones o asume roles distintos, Captain utiliza corchetes para especificar el contexto. En nuestro caso, no diremos simplemente "Un estudiante debe estar asignado a un usuario", sino que precisaremos: "Un `estudiante` debe ser autenticado por solo un `usuario` `[inicio de sesión]`". Del mismo modo, para la relación unaria de creación de cuentas, especificaremos: "Cada `usuario` debe ser creado por solo un `[otro]` `usuario`".
2.  **La regla lingüística: Plural (Clase) vs. Singular (Ocurrencia):** Al redactar en lenguaje natural, el uso del número gramatical no es accidental. El plural (`cursos`) se utiliza para referirse a la Clase de Entidad (el concepto colectivo o catálogo general). El singular (`clase_programada`) se utiliza para referirse a una Ocurrencia de Entidad (una instancia específica y tangible). *Ejemplo:* Decimos "Un `curso` puede ser materializado muchas veces" (Clase), pero "Cada `clase_programada` debe ser impartida por un único `docente`" (Ocurrencia).
3.  **El "Matiz Oculto" como herramienta de entrevista:** La obligatoriedad `0` no es solo teoría; es una pregunta de validación directa para el cliente: "Si la obligatoriedad es 'puede' (0), ¿existe la posibilidad real de que esta relación nunca suceda?" (Ej. ¿Es posible que un docente esté en el sistema y que jamás imparta una clase?).

Habiendo establecido los cimientos teóricos y conceptuales que rigen la lógica de las relaciones, no basta con comprender la teoría; es imperativo materializarla. El tránsito de la abstracción a la praxis exige que traduzcamos estos conceptos en enunciados formales. Por ello, antes de visualizar el panorama completo en una matriz, debemos dominar la sintaxis de la afirmación individual, átomo fundamental de nuestra validación.

##### 4.2 Paso 4-1: Redacción formal de las afirmaciones

Procedemos a redactar las afirmaciones en texto completo siguiendo la sintaxis estricta de Captain: *Ocurrencia de Entidad + Obligatoriedad + Relación + Cardinalidad + Ocurrencia/Clase de Entidad*. Cada relación produce dos afirmaciones. Utilizamos la palabra "**Un/Una**" para la dirección A → B, y "**Cada**" para la dirección B → A.

**Afirmaciones del Dominio Operativo (Backend)**

| Relación | Dirección A → B (Inicia con "Un/Una") | Dirección B → A (Inicia con "Cada") |
| :--- | :--- | :--- |
| `usuarios` ↔ `usuarios` (`crear`) | Un `usuario` **puede** crear muchos `usuarios` (0:N) | Cada `usuario` **debe** ser creado por solo un `[otro]` `usuario` (1:1) |
| `niveles_acceso` ↔ `usuarios` (`autorizar`) | Un `nivel_acceso` **puede** autorizar a muchos `usuarios` (0:N) | Cada `usuario` **debe** tener autorizado solo un `nivel_acceso` (1:1) |
| `registros_actividad` ↔ `usuarios` (`generar`) | Un `registro_actividad` **debe** ser generado por solo un `usuario` (1:1) | Cada `usuario` **puede** generar muchos `registros_actividad` (0:N) |

**Afirmaciones de los Puentes de Identidad (Frontend ↔ Backend)**

| Relación | Dirección A → B (Inicia con "Un/Una") | Dirección B → A (Inicia con "Cada") |
| :--- | :--- | :--- |
| `usuarios` ↔ `estudiantes` (`autenticar`) | Un `estudiante` **debe** ser autenticado por solo un `usuario` `[inicio de sesión]` (1:1) | Cada `usuario` `[inicio de sesión]` **puede** autenticar a solo un `estudiante` (0:1) |
| `usuarios` ↔ `docentes` (`autenticar`) | Un `docente` **debe** ser autenticado por solo un `usuario` `[inicio de sesión]` (1:1) | Cada `usuario` `[inicio de sesión]` **puede** autenticar a solo un `docente` (0:1) |

**Afirmaciones de la Lógica Académica (Frontend)**

| Relación | Dirección A → B (Inicia con "Un/Una") | Dirección B → A (Inicia con "Cada") |
| :--- | :--- | :--- |
| `cursos` ↔ `cursos` (`ser_prerrequisito`) | Un `curso` **puede** ser prerrequisito de muchos `[otros]` `cursos` (0:N) | Cada `curso` **puede** tener como prerrequisitos a muchos `[otros]` `cursos` (0:N) |
| `docentes` ↔ `cursos` (`impartir`) | Un `docente` **puede** impartir muchos `cursos` (0:N) | Cada `curso` **puede** ser impartido por muchos `docentes` (0:N) |
| `docentes` ↔ `clases_programadas` (`asignar`) | Un `docente` **puede** tener asignadas muchas `clases_programadas` (0:N) | Cada `clase_programada` **debe** ser asignada a solo un `docente` (1:1) |
| `cursos` ↔ `clases_programadas` (`materializar`) | Un `curso` **puede** ser materializado en muchas `clases_programadas` (0:N) | Cada `clase_programada` **debe** materializar a solo un `curso` (1:1) |
| `semestres` ↔ `clases_programadas` (`pertenecer`) | Un `semestre` **puede** contener muchas `clases_programadas` (0:N) | Cada `clase_programada` **debe** pertenecer a solo un `semestre` (1:1) |
| `estudiantes` ↔ `clases_programadas` (`inscribir`) | Un `estudiante` **puede** inscribirse en muchas `clases_programadas` (0:N) | Cada `clase_programada` **puede** ser inscrita por muchos `estudiantes` (0:N) |

Si bien la redacción textual de cada afirmación constituye el átomo de nuestra validación semántica, presentar al cliente una lista extensa y fragmentada de oraciones puede resultar abrumador y poco práctico para una revisión ágil. Aquí es donde la ingeniería de software debe ceder el paso a la ergonomía cognitiva. Por ello, una vez dominada la sintaxis individual, es pertinente agrupar estas reglas en una estructura de síntesis: la Matriz Consolidada. Esta herramienta no es un mero resumen, sino un artefacto de validación holística que permite visualizar la lógica de negocio bidireccional de un solo vistazo, facilitando la detección de inconsistencias antes de la formalización final.

##### 4.3 La Matriz Consolidada

La Matriz Consolidada de Obligatoriedad y Cardinalidad que se presenta a continuación no forma parte de la metodología original de Fidel A. Captain, quien pasa directamente de la Matriz E-E (Paso 2) a la redacción textual de las afirmaciones (Paso 4). Sin embargo, en la práctica profesional del modelado de datos, esta matriz consolidada resulta una herramienta sumamente pertinente y poderosa. Permite visualizar la lógica de negocio completa en un solo vistazo, condensando el verbo relacional con sus restricciones bidireccionales. Esto facilita enormemente las sesiones de validación con el cliente, permitiéndole leer y aprobar la lógica de negocio antes de pasar a la redacción formal en texto completo.

A continuación, presentamos la Matriz Consolidada para este estudio de caso. En cada celda, el formato es: `Verbo (Obligatoriedad:Cardinalidad Fila→Columna / Obligatoriedad:Cardinalidad Columna→Fila)`.

**Matriz Consolidada de Relaciones**

| | `estudiantes` | `cursos` | `clases_programadas` | `semestres` | `docentes` | `usuarios` | `niveles_acceso` | `registros_actividad` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`estudiantes`** | — | | se inscriben en<br>(0:N / 0:N) | | | son autenticados por<br>(1:1 / 0:1) | | |
| **`cursos`** | | \* son prerrequisito de<br>(0:N / 0:N) | son materializados en<br>(0:N / 1:1) | | son impartidos por<br>(0:N / 0:N) | | | |
| **`clases_programadas`** | | | — | pertenecen a<br>(1:1 / 0:N) | se asignan a<br>(1:1 / 0:N) | | | |
| **`semestres`** | | | | — | | | | |
| **`docentes`** | | | | | — | son autenticados por<br>(1:1 / 0:1) | | |
| **`usuarios`** | | | | | | \* crean<br>(0:N / 1:1) | autorizan a<br>(1:1 / 0:N) | generan<br>(0:N / 1:1) |
| **`niveles_acceso`** | | | | | | | — | |
| **`registros_actividad`** | | | | | | | | — |

*(Nota: Las celdas con `*` en la diagonal representan Relaciones Unarias).*

**Cómo leer esta matriz con el cliente:** Si tomamos la intersección de `docentes` (Fila) y `clases_programadas` (Columna), leemos: "Los `docentes` pueden tener asignadas (0) muchas (N) `clases_programadas`, pero cada `clase_programada` debe (1) ser asignada a solo un (1) `docente`". Esta lectura directa y compacta es ideal para detectar errores de lógica de negocio en minutos.

Con la Matriz Consolidada estructurada, el modelo deja de ser un ejercicio solitario del diseñador y se convierte en un contrato operativo sujeto a escrutinio. La verdadera prueba de fuego no reside en la corrección gráfica de la matriz, sino en su capacidad para revelar las ambigüedades ocultas durante el diálogo con el cliente. Es en este espacio de validación donde el "matiz oculto del cero" se erige como nuestra sonda metodológica más valiosa.

##### 4.4 Validación y verificación con el cliente

Las afirmaciones y la Matriz Consolidada funcionan como contratos operativos. Leerlas en voz alta con el cliente revela ambigüedades ocultas que la mera inspección visual del diagrama no puede detectar. A continuación, se presentan ejemplos de cómo utilizar el "matiz oculto del cero" para validar la lógica de negocio:

- **Ejemplo 1: Opcionalidad en la contratación docente.** La afirmación «Un `docente` puede tener asignadas muchas `clases_programadas` (0:N)» admite que un docente recién contratado aún no tenga clases asignadas. La obligatoriedad `0` refleja la realidad de los procesos de contratación académica: existe la posibilidad de que un docente no pueda impartir ninguna clase en un semestre determinado (ej. año sabático, licencia médica). *Pregunta al cliente:* "¿Es posible que un docente esté en el sistema sin impartir ninguna clase?" Si la respuesta es sí, la obligatoriedad `0` es correcta.
- **Ejemplo 2: Trazabilidad de registros de actividad.** La afirmación «Cada `registro_actividad` debe ser generado por solo un `usuario` (1:1)» garantiza que cada evento de autenticación tenga un responsable único. La obligatoriedad `1` es imperativa: no puede existir un registro de actividad sin un usuario que lo genere. En la base de datos, la columna `id_usuario` en la tabla `registros_actividad` será estrictamente `NOT NULL`.
- **Ejemplo 3: Flexibilidad curricular en prerrequisitos.** La relación unaria `cursos` ↔ `cursos` es simétrica (0:N en ambas direcciones). Un curso puede no tener prerrequisitos (ej. cursos introductorios) y puede ser prerrequisito de múltiples asignaturas superiores. La obligatoriedad `0` en ambas direcciones refleja esta flexibilidad curricular sin forzar dependencias inexistentes.

Habiendo sometido las reglas de negocio al crisol de la validación con el cliente y resuelto las ambigüedades semánticas, nos encontramos en el umbral de la siguiente fase. Antes de dar por concluida esta etapa y proceder a la inyección de estas reglas en el diagrama, es imperativo aplicar un filtro de rigor metodológico. Este checklist actúa como nuestro último guardián metodológico del Paso 4.

##### 4.5 Lista de validación

Antes de avanzar al Paso 5 (Creación del diagrama E-R detallado), verifique que se cumplan las siguientes condiciones. Actúa como nuestro filtro metodológico final para esta fase:

- ✅ ¿Cada relación del diagrama E-R simplificado tiene exactamente dos afirmaciones (una en cada dirección)?
- ✅ ¿Las afirmaciones siguen la sintaxis estricta: Ocurrencia de Entidad + Obligatoriedad + Relación + Cardinalidad + Ocurrencia/Clase de Entidad?
- ✅ ¿Se utiliza "Un/Una" para la dirección A→B y "Cada" para la dirección B→A?
- ✅ ¿Las obligatoriedades (`0` o `1`) reflejan reglas de negocio verificables con el cliente y se corresponden correctamente con la restricción física `NULL` / `NOT NULL`?
- ✅ ¿Las cardinalidades (`1` o `N`) están justificadas por la naturaleza del dominio?
- ✅ ¿El cliente ha verificado y aprobado tanto la Matriz Consolidada como cada afirmación textual?

Si todas las respuestas son afirmativas, el modelo conceptual está listo para evolucionar. En el siguiente paso, inyectaremos estas afirmaciones en el diagrama E-R simplificado mediante la notación `Obligatoriedad:Cardinalidad`, transformando esta estructura estática en un modelo dinámico, validado y preparado para su traducción al lenguaje de implementación. Transitaremos, así, de la semántica operativa a la topología detallada.

---

#### Paso 5: Diagrama Entidad-Relación detallado: La inyección de restricciones dinámicas

Una vez que hemos definido con precisión las reglas de negocio en el Paso 4, es momento de integrarlas visualmente en nuestro modelo. El **Paso 5** tiene un propósito claro y mecánico, pero de una profundidad conceptual mayúscula: fusionar el diagrama Entidad-Relación simplificado (del Paso 3) con las afirmaciones textuales (del Paso 4) para generar un diagrama E-R detallado. 

Como señala Fidel A. Captain (2015, p. 78): «Este *diagrama E-R detallado* proporciona un *modelo conceptual completo de los datos*, uno que representa la visión del usuario de los datos y la estructura lógica de la base de datos». En mi opinión, este paso constituye el puente definitivo entre la estructura estática de las entidades y la praxeología dinámica de las reglas de negocio, preparando el terreno para la futura implementación física.

##### 5.1 Fundamento conceptual y notación

En esta etapa, las afirmaciones ya no se escriben como oraciones completas en el diagrama, sino que se condensan en su forma más pura y técnica: la notación **Obligatoriedad:Cardinalidad (O:C)**. 

La convención establecida por Captain es colocar esta notación en la línea que conecta la entidad (rectángulo) con la relación (rombo), que nosotros hemos sustituido por una línea. La regla de oro para su ubicación es sintáctica y no arbitraria: **cada afirmación siempre se coloca más cerca de la primera ocurrencia de entidad enunciada en la afirmación**. Esto garantiza una lectura unívoca y evita la ambigüedad semántica.

Las combinaciones posibles que aparecerán en nuestro diagrama, derivadas directamente de la lógica del negocio, son:  
- **`0:1`**: Puede relacionarse con solo uno (opcional y único).  
- **`0:N`**: Puede relacionarse con muchos (opcional y múltiple).  
- **`1:1`**: Debe relacionarse con solo uno (obligatorio y único).  
- **`1:N`**: Debe relacionarse con muchos (obligatorio y múltiple).  
- **`M:N`**: Muchos a muchos. Una o muchas ocurrencias de una entidad pueden relacionarse con una o muchas ocurrencias de la otra entidad. En el modelo conceptual, esto se representa mediante un rombo central conectado a ambas entidades, reflejando una intersección conceptual de mutua contingencia donde ninguna de las dos entidades subsume a la otra. Como veremos en el Paso 6, esta relación no puede materializarse directamente en el modelo físico y requerirá la creación de una tabla de unión (entidad asociativa) para preservar la integridad referencial.

#### 5.2 Paso 5-1 según Captain

Para construir el diagrama detallado de manera sistemática y verificable, seguimos dos acciones secuenciales:
1.  **Preparar la lista de afirmaciones con notación (O:C):** Tomamos las afirmaciones validadas en el Paso 4 y añadimos explícitamente el par `Obligatoriedad:Cardinalidad` al final de cada una, consolidando la lógica en una sola línea.  
2.  **Insertar las notaciones en el diagrama:** Recorremos el diagrama simplificado relación por relación. Para cada vínculo, colocamos los dos valores `O:C` en sus respectivos lados, asegurándonos de que el valor correspondiente a la dirección A → B quede junto a la Entidad A, y el de la dirección B → A quede junto a la Entidad B.  

> **Recuerda:** *En el diagrama E-R detallado, cada relación tiene dos afirmaciones asociadas, una en cada lado de la relación. Además, cada afirmación siempre se coloca más cerca de la primera ocurrencia de entidad enunciada en la afirmación.*  (Captain, 2015, p. 81).

##### 5.3 Aplicación al Estudio de Caso

A continuación, aplicamos este procedimiento a nuestro sistema académico, agrupando las relaciones por dominios para mantener la claridad arquitectónica que hemos venido desarrollando. *(Nota: Se utilizan los nombres de entidad en español y notación snake_case, consistentes con el modelo conceptual definido en los pasos anteriores).*

**1. Relaciones Intra-Backend (Capa de Seguridad)**

| Relación | Afirmación A → B (con O:C) | Afirmación B → A (con O:C) | Representación en el Diagrama |
| :--- | :--- | :--- | :--- |
| `usuarios` ↔ `usuarios` (`crear`) | Un `usuario` **puede** crear muchos `usuarios` **(0:N)** | Cada `usuario` **debe** ser creado por solo un `[otro]` `usuario` **(1:1)** | `[usuarios] ── 0:N <crear> 1:1 ── [usuarios]*` |
| `niveles_acceso` ↔ `usuarios` (`autorizar`) | Un `nivel_acceso` **puede** autorizar a muchos `usuarios` **(0:N)** | Cada `usuario` **debe** tener autorizado solo un `nivel_acceso` **(1:1)** | `[niveles_acceso] ── 0:N <autorizar> 1:1 ── [usuarios]` |
| `registros_actividad` ↔ `usuarios` (`generar`) | Un `registro_actividad` **debe** ser generado por solo un `usuario` **(1:1)** | Cada `usuario` **puede** generar muchos `registros_actividad` **(0:N)** | `[usuarios] ── 0:N <generar> 1:1 ── [registros_actividad]` |

**2. Relaciones Puente (Frontend ↔ Backend)**

| Relación | Afirmación A → B (con O:C) | Afirmación B → A (con O:C) | Representación en el Diagrama |
| :--- | :--- | :--- | :--- |
| `usuarios` ↔ `estudiantes` (`autenticar`) | Un `estudiante` **debe** ser autenticado por solo un `usuario` `[inicio de sesión]` **(1:1)** | Cada `usuario` `[inicio de sesión]` **puede** autenticar a solo un `estudiante` **(0:1)** | `[estudiantes] ── 1:1 <autenticar> 0:1 ── [usuarios]` |
| `usuarios` ↔ `docentes` (`autenticar`) | Un `docente` **debe** ser autenticado por solo un `usuario` `[inicio de sesión]` **(1:1)** | Cada `usuario` `[inicio de sesión]` **puede** autenticar a solo un `docente` **(0:1)** | `[docentes] ── 1:1 <autenticar> 0:1 ── [usuarios]` |

>*Nota:* Observa cómo en los puentes, la obligatoriedad `1` recae sobre el perfil académico (`estudiante`/`docente`), mientras que la cuenta de `usuario` tiene una opcionalidad `0`. Esto no es un error, sino una decisión de diseño que permite la existencia de usuarios administrativos no vinculados a un perfil académico específico.

**3. Relaciones Intra-Frontend (Lógica Académica)**

| Relación | Afirmación A → B (con O:C) | Afirmación B → A (con O:C) | Representación en el Diagrama |
| :--- | :--- | :--- | :--- |
| `cursos` ↔ `cursos` (`ser_prerrequisito`) | Un `curso` **puede** ser prerrequisito de muchos `[otros]` `cursos` **(0:N)** | Cada `curso` **puede** tener como prerrequisitos a muchos `[otros]` `cursos` **(0:N)** | `[cursos] ── 0:N <ser_prerrequisito> 0:N ── [cursos]*` |
| `docentes` ↔ `cursos` (`impartir`) | Un `docente` **puede** impartir muchos `cursos` **(0:N)** | Cada `curso` **puede** ser impartido por muchos `docentes` **(0:N)** | `[docentes] ── 0:N <impartir> 0:N ── [cursos]` |
| `docentes` ↔ `clases_programadas` (`asignar`) | Un `docente` **puede** tener asignadas muchas `clases_programadas` **(0:N)** | Cada `clase_programada` **debe** ser asignada a solo un `docente` **(1:1)** | `[docentes] ── 0:N <asignar> 1:1 ── [clases_programadas]` |
| `cursos` ↔ `clases_programadas` (`materializar`) | Un `curso` **puede** ser materializado en muchas `clases_programadas` **(0:N)** | Cada `clase_programada` **debe** materializar a solo un `curso` **(1:1)** | `[cursos] ── 0:N <materializar> 1:1 ── [clases_programadas]` |
| `semestres` ↔ `clases_programadas` (`pertenecer`) | Un `semestre` **puede** contener muchas `clases_programadas` **(0:N)** | Cada `clase_programada` **debe** pertenecer a solo un `semestre` **(1:1)** | `[semestres] ── 0:N <pertenecer> 1:1 ── [clases_programadas]` |
| `estudiantes` ↔ `clases_programadas` (`inscribir`) | Un `estudiante` **puede** inscribirse en muchas `clases_programadas` **(0:N)** | Cada `clase_programada` **puede** ser inscrita por muchos `estudiantes` **(0:N)** | `[estudiantes] ── 0:N <inscribir> 0:N ── [clases_programadas]` |

##### 5.4 Topología del Diagrama E-R Detallado

Al ensamblar todos los fragmentos anteriores, incluyendo las relaciones `M:N` con sus notaciones duales, obtenemos la visión completa y detallada del modelo conceptual. La notación `*` indica la duplicación visual de una entidad para evitar cruces de líneas confusos en las relaciones unarias, tal como lo recomienda Captain, priorizando la legibilidad sobre el formalismo estricto.

```text
[niveles_acceso] ── 0:N <autorizar> ── 1:1 ── [usuarios] ── 0:N <crear> ── 1:1 ── [usuarios]*
                                              │
                                              ├── 0:1 <autenticar> ── 1:1 ── [estudiantes]
                                              ├── 0:1 <autenticar> ── 1:1 ── [docentes]
                                              └─ 1:1 <generar> ── 0:N ── [registros_actividad]

[semestres] ── 0:N <pertenecer> ── 1:1 ── [clases_programadas] ── 1:1 <asignar> ── 0:N ── [docentes]
                                                    │                        │
                                                    │                        └── 0:N <impartir> ── 0:N ── [cursos]
                                                    │
                                                    ├── 1:1 <materializar> ── 0:N ── [cursos] ── 0:N <ser_prerrequisito> ── 0:N ── [cursos]*
                                                    │
                                                    └─ 0:N <inscribir> ── 0:N ── [estudiantes]
```
*(Nota de lectura: Para interpretar una conexión, lee desde la entidad de origen, toma el primer par `O:C`, lee el verbo de la relación, toma el segundo par `O:C` y llega a la entidad de destino. Ejemplo: `[semestres]` (0:N) `<pertenecer>` (1:1) `[clases_programadas]`).*

##### 5.5 Checklist de validación pre-Paso 6

Antes de dar por concluido este paso y avanzar hacia la transformación al modelo de implementación (Paso 6), Captain nos insta a realizar una verificación cruzada final. Este checklist actúa como nuestro último filtro metodológico del modelado conceptual:

- ✅ ¿Cada relación en el diagrama tiene exactamente dos notaciones `O:C`, una a cada lado del rombo (incluyendo las relaciones `M:N`)?
- ✅ ¿La notación `O:C` colocada junto a una entidad corresponde exactamente a la *primera* entidad mencionada en la afirmación textual del Paso 4?
- ✅ ¿Las obligatoriedades (`0` o `1`) reflejan fielmente las reglas de negocio validadas con el cliente?
- ✅ ¿Las cardinalidades (`1` o `N`) están justificadas por la naturaleza del dominio (ej. una `clase_programada` *debe* tener un solo `docente`, pero un `docente` *puede* tener muchas `clases_programadas`)?
- ✅ ¿El diagrama es legible y las relaciones unarias están representadas sin generar ambigüedad visual?

Si todas las respuestas son afirmativas, hemos completado con éxito la fase de modelado conceptual. El diagrama E-R detallado está ahora maduro, validado y listo para ser traducido al lenguaje del desarrollador en el **Paso 6: Transformar el diagrama E-R detallado en un diagrama R-M implementable**, donde los rombos desaparecerán para dar paso a las claves foráneas y la notación de Pata de Gallo (*Crow's Foot*), cimentando la integridad de los datos en la estructura física.

---

#### Fase III: Implementación Lógica (Paso 6)

Cruzar el umbral de esta fase implica un cambio de visión radical. El modelo conceptual, fiel a la realidad del negocio, debe ahora someterse a las rigideces y convenciones de la máquina. Aquí los rombos colapsan para dar paso a las claves foráneas y la normalización, materializando la integridad referencial en un esquema SQL listo para su despliegue en un Sistema Gestor de Bases de Datos Relacionales (RDBMS).

#### Paso 6: Transformar el diagrama E-R detallado en un diagrama R-M implementable

Este es el sexto y último paso del proceso de diseño conceptual. Como señala Fidel A. Captain (2015, p. 94): *«Transformar un diagrama E-R en un diagrama R-M no es solo un cambio de diagramas, sino también un cambio de visión. El diagrama E-R es un modelo conceptual y representa la visión del usuario de los datos [...] mientras que el diagrama R-M es un modelo de implementación y representa la visión del desarrollador de los datos y la estructura física de la base de datos»*.

**El cambio de paradigma lingüístico y técnico:** 

Hasta el Paso 5, nos mantuvimos fieles al *orgware* del cliente, modelando en español para facilitar la validación semántica. Sin embargo, al cruzar el umbral del Paso 6, ocurre una transformación deliberada: la traducción al lenguaje universal de los Sistemas Gestores de Bases de Datos (RDBMS). Las entidades y atributos mutan a su contraparte técnica en inglés, adoptando estrictamente la notación **`snake_case`** (minúsculas separadas por guiones bajos) y la forma **singular**. Esta decisión no es caprichosa; es una medida pragmática para garantizar la compatibilidad nativa, la eficiencia de los índices y la precisión semántica universal, evitando la "jaula de hierro" técnica que imponen los espacios, las mayúsculas o los caracteres especiales en los scripts SQL.

En esta fase, las entidades se convierten en tablas bidimensionales y los vínculos se cimentan mediante la migración de Claves Primarias (PK) que se transforman en Claves Foráneas (FK, de *Foreign Key*), sellando la integridad referencial mediante la notación de Pata de Gallo (*Crow's Foot*).

**Normalización**

La metodología de los Seis Pasos está diseñada para que, al aplicar sus reglas de transformación, el modelo cumpla automáticamente con la **Tercera Forma Normal (3FN)**, sin necesidad de un proceso correctivo posterior:

1. **1NF (Integridad de Dominio):** Garantizada desde el Paso 1 (atributos atómicos, sin grupos repetitivos, PK definida).  
2. **2NF (Integridad de Entidad):** Asegurada en el **Paso 6.1**. Al crear tablas de unión para relaciones N:N, se garantiza que los atributos relacionales dependan de la totalidad de la clave.  
3. **3NF (Integridad Referencial):** Asegurada en los **Pasos 6.2 y 6.3**. Al colocar Claves Foráneas (FK) en lugar de duplicar datos descriptivos, se eliminan las dependencias transitivas.  

El proceso se realiza de manera iterativa, abordando cada tipo de cardinalidad por separado para mantener la integridad del modelo.

##### 6.1 Transformar Relaciones Muchos a Muchos (N:N)

-   **Regla de Transformación:** Se elimina el rombo. Se crea una *nueva Relación* (tabla de unión). Las claves primarias de las dos entidades originales se convierten en claves foráneas (FK) en esta nueva tabla, y se crea una nueva Clave Primaria (PK) subrogada independiente.  
-   **Aclaración:** Una clave subrogada (*surrogate key*) es un identificador numérico generado por el sistema, carente de significado en el mundo real del negocio. Se utiliza porque es computacionalmente más eficiente para los índices y protege al sistema de la volatilidad de los códigos institucionales.  
-   **Aplicación al Caso:** Se crean las tablas `student_class`, `course_prerequisite` y `course_lecturer`, trasladando sus atributos relacionales (`registered_date`, `min_pass_grade`) a estas nuevas estructuras.  

##### 6.2 Transformar Relaciones Uno a Muchos (1:N)

-   **Regla de Transformación:** La entidad del lado "N" **absorbe** la relación.  
-   **Aclaración:** En la praxeología de Captain, "absorber" significa que una tabla incorpora físicamente la PK de la otra tabla como una nueva FK. La tabla que "absorbe" crece en columnas; la otra permanece intacta.  
-   **Aplicación al Caso:** `scheduled_class` absorbe las FK `semester_id`, `course_id` y `lecturer_id`.  

##### 6.3 Transformar Relaciones Uno a Uno (1:1)

-   **Regla de Transformación:** La entidad que tiene una obligatoriedad de `1` (debe) absorbe la relación. La PK de la entidad no absorbente se convierte en una FK dentro de la absorbente.  
-   **Aplicación al Caso:** `student` y `lecturer` absorben `user_id`. `app_user` absorbe `access_level_id`. `log_entry` absorbe `user_id`. `app_user` absorbe `creator_user_id` (para la relación unaria de auditoría).  

##### 6.4: Combinar Resultados para el Diagrama del Modelo Relacional Final

-   **Regla de Transformación:** Se unen todos los diagramas parciales para formar el diagrama del modelo relacional final, verificando que todas las FK apunten correctamente a sus PK de origen, estableciendo una jerarquía de creación de tablas.  

#### 6.5 Lógica de Integridad, Índices y Vistas (Conceptos Pre-Implementación)

Antes de definir el diccionario de datos, es crucial entender la lógica detrás de las restricciones físicas que protegerán el modelo:

1. **Preservación de Integridad en Cascada:**   
   - `ON UPDATE CASCADE`: Garantiza que si una PK cambia, todas las FK se actualicen, evitando registros huérfanos.  
   - `ON DELETE RESTRICT`: Vital para datos históricos. Impide que se elimine un `course` o un `app_user` si existen registros de actividad o inscripciones asociadas, protegiendo la trazabilidad académica.  
2. **Lógica de los Índices:** Los motores de RDBMS indexan automáticamente las PK. Declararemos índices explícitos en columnas de búsqueda frecuente (`email`, `last_name`) para optimizar las cláusulas `WHERE`.  
3. **Lógica y Utilidad de las Vistas (`VIEW`):** Son consultas SQL almacenadas que actúan como tablas virtuales. En la capa de aplicación, ocultan la complejidad de los `JOIN` y restringen el acceso a columnas sensibles (ej. una vista `vw_student_dashboard` que une 6 tablas pero solo expone nombres y horarios, ocultando `password` o `ss_number`).  

##### 6.6 Diccionario de Datos Pre-Implementación

> **Nota Metodológica:** La presentación detallada de un Diccionario de Datos con tipos abstractos ANSI no forma parte explícita de la metodología original de Captain. Sin embargo, se introduce aquí como una práctica profesional indispensable de prevalidación para garantizar la independencia de plataforma y la robustez estructural antes de la implementación física.

A continuación, se presenta el desglose de las 11 tablas con su estructura, tipos abstractos y restricciones, numeradas consecutivamente según su orden topológico de creación, utilizando estrictamente la notación `snake_case` en singular.

**1. Tabla `access_level`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `access_level_id` | `INTEGER` | PK, Identity |
| `access_level_code` | `VARCHAR(10)` | NOT NULL, UNIQUE |
| `short_name` | `VARCHAR(50)` | NOT NULL |
| `long_name` | `VARCHAR(100)` | |
| `access_level_description` | `TEXT` | |

**2. Tabla `semester`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `semester_id` | `INTEGER` | PK, Identity |
| `academic_cycle` | `VARCHAR(9)` | NOT NULL |
| `semester_term` | `SMALLINT` | NOT NULL |
| `start_date` | `DATE` | NOT NULL |
| `end_date` | `DATE` | NOT NULL |

**3. Tabla `course`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `course_id` | `INTEGER` | PK, Identity |
| `course_code` | `VARCHAR(10)` | NOT NULL, UNIQUE |
| `short_name` | `VARCHAR(50)` | NOT NULL |
| `long_name` | `VARCHAR(100)` | |
| `course_description` | `TEXT` | |

**4. Tabla `app_user`** *(Nota: se usa `app_user` en lugar de `user` para evitar conflictos con la palabra reservada estándar de SQL)*

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `user_id` | `INTEGER` | PK, Identity |
| `login` | `VARCHAR(50)` | NOT NULL, UNIQUE |
| `user_name` | `VARCHAR(100)` | NOT NULL |
| `password` | `VARCHAR(255)` | NOT NULL |
| `is_active` | `BOOLEAN` | NOT NULL, DEFAULT TRUE |
| `access_level_id` | `INTEGER` | FK, NOT NULL |
| `creator_user_id` | `INTEGER` | FK, NOT NULL |

**5. Tabla `student`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `student_id` | `INTEGER` | PK, Identity |
| `ss_number` | `CHAR(11)` | NOT NULL, UNIQUE |
| `last_name` | `VARCHAR(50)` | NOT NULL |
| `first_name` | `VARCHAR(50)` | NOT NULL |
| `middle_name` | `VARCHAR(50)` | |
| `gender` | `VARCHAR(6)` | NOT NULL, DEFAULT 'MALE' |
| `dob` | `DATE` | NOT NULL |
| `email` | `VARCHAR(100)` | NOT NULL, UNIQUE |
| `mobile` | `VARCHAR(15)` | |
| `htel` | `VARCHAR(15)` | NOT NULL |
| `wtel` | `VARCHAR(15)` | |
| `address_line1` | `VARCHAR(100)` | NOT NULL |
| `address_line2` | `VARCHAR(100)` | |
| `city` | `VARCHAR(50)` | NOT NULL |
| `state` | `VARCHAR(50)` | NOT NULL |
| `postal_code` | `VARCHAR(10)` | NOT NULL |
| `user_id` | `INTEGER` | FK, NOT NULL |

**6. Tabla `lecturer`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `lecturer_id` | `INTEGER` | PK, Identity |
| `employee_number` | `VARCHAR(20)` | NOT NULL, UNIQUE |
| `ss_number` | `CHAR(11)` | NOT NULL, UNIQUE |
| `last_name` | `VARCHAR(50)` | NOT NULL |
| `first_name` | `VARCHAR(50)` | NOT NULL |
| `middle_name` | `VARCHAR(50)` | |
| `gender` | `VARCHAR(6)` | NOT NULL, DEFAULT 'MALE' |
| `email` | `VARCHAR(100)` | NOT NULL, UNIQUE |
| `mobile` | `VARCHAR(15)` | |
| `htel` | `VARCHAR(15)` | |
| `wtel` | `VARCHAR(15)` | |
| `about` | `TEXT` | |
| `user_id` | `INTEGER` | FK, NOT NULL |

**7. Tabla `log_entry`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `log_entry_id` | `BIGINT` | PK, Identity |
| `start_datetime` | `TIMESTAMP` | NOT NULL |
| `end_datetime` | `TIMESTAMP` | |
| `table_name` | `VARCHAR(50)` | NOT NULL |
| `table_pk` | `VARCHAR(50)` | NOT NULL |
| `change_type` | `VARCHAR(20)` | NOT NULL |
| `details` | `TEXT` | |
| `user_id` | `INTEGER` | FK, NOT NULL |

**8. Tabla `scheduled_class`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `class_id` | `BIGINT` | PK, Identity |
| `schedule_code` | `VARCHAR(20)` | NOT NULL, UNIQUE |
| `section` | `CHAR(1)` | NOT NULL |
| `day` | `VARCHAR(10)` | NOT NULL |
| `time` | `TIME` | NOT NULL |
| `location` | `VARCHAR(75)` | |
| `semester_id` | `INTEGER` | FK, NOT NULL |
| `course_id` | `INTEGER` | FK, NOT NULL |
| `lecturer_id` | `INTEGER` | FK, NOT NULL |

**9. Tabla `course_prerequisite`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `course_prerequisite_id` | `INTEGER` | PK, Identity |
| `min_pass_grade` | `VARCHAR(2)` | |
| `course_id` | `INTEGER` | FK, NOT NULL |
| `prereq_course_id` | `INTEGER` | FK, NOT NULL |

**10. Tabla `course_lecturer`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `course_lecturer_id` | `INTEGER` | PK, Identity |
| `course_id` | `INTEGER` | FK, NOT NULL |
| `lecturer_id` | `INTEGER` | FK, NOT NULL |

**11. Tabla `student_class`**

| Atributo | Tipo ANSI | Restricciones |
| :--- | :--- | :--- |
| `student_class_id` | `INTEGER` | PK, Identity |
| `registered_date` | `TIMESTAMP` | NOT NULL |
| `student_id` | `INTEGER` | FK, NOT NULL |
| `class_id` | `BIGINT` | FK, NOT NULL |

##### 6.7 Reglas de Integridad y de Negocio (Prevalidación)

*(Se mantienen las reglas de Entidad, Dominio, Referencial y Negocio definidas previamente, ahora materializadas en las restricciones `CHECK` y `UNIQUE` del diccionario).*

##### 6.8 La Lógica del Orden Topológico de Creación

Para que un RDBMS pueda crear las tablas y establecer las Claves Foráneas (FK), debe existir una regla inquebrantable: **una tabla no puede hacer referencia a otra tabla que aún no existe**. 

A esto se le llama **orden topológico de creación**. Imagina que estás en una oficina de registro civil: no puedes registrar el acta de nacimiento de un bebé (Tabla Hija / Dependiente) si primero no has registrado los identificadores oficiales de sus padres (Tabla Padre / Independiente). El sistema simplemente rechazará el trámite.

El orden es estricto:  
1. **Primero:** Tablas Independientes (sin FK).  
2. **Segundo:** Tablas Dependientes (con FK que apuntan a las independientes).  
3. **Tercero:** Tablas de Unión (con FK que apuntan a dos o más tablas ya creadas).  

**Contraejemplo: Las consecuencias de ignorar el orden topológico**

Si un desarrollador intenta crear el script en un orden aleatorio o alfabético, el RDBMS lanzará un error fatal de "Restricción de Clave Foránea" (*Foreign Key Constraint Fails*) y abortará la creación de la base de datos.

```sql
-- ❌ SCRIPT CON ORDEN INCORRECTO (FALLARÁ)

-- Intentamos crear la tabla dependiente primero
CREATE TABLE student (
  student_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id INTEGER,
  -- ¡ERROR FATAL! El RDBMS busca la tabla 'app_user' para validar la FK, 
  -- pero 'app_user' aún no ha sido creada.
  FOREIGN KEY (user_id) REFERENCES app_user(user_id) 
);

-- La tabla independiente se crea después (demasiado tarde)
CREATE TABLE app_user (
  user_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  login VARCHAR(50) UNIQUE
);
```

**Solución:** Invertir el orden. Primero se crea `app_user`, y solo después `student` puede referenciarla con seguridad.

---

##### 6.9 Implementación Técnica: Script SQL Estándar (SQL-92)

A continuación, se presenta la traducción del diagrama del modelo relacional a un script de implementación siguiendo el estándar ANSI SQL-92 (cf. Digital Equipment Corporation, 1993). 

> **Nota de Refinamiento Profesional:** A diferencia de transcripciones ingenuas que usan sintaxis propietaria (como `AUTO_INCREMENT` o `int(11)`), este script ha sido depurado y estandarizado. Utiliza `GENERATED ALWAYS AS IDENTITY` para claves subrogadas, tipos de datos universales y separa la creación de índices en sentencias posteriores, garantizando que este código sea portable y ejecutable en cualquier motor de base de datos moderno.

```sql
-- =================================================================
-- 1. TABLAS INDEPENDIENTES (Sin Claves Foráneas)
-- =================================================================

CREATE TABLE access_level (
  access_level_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  access_level_code VARCHAR(10) NOT NULL UNIQUE,
  short_name VARCHAR(50) NOT NULL,
  long_name VARCHAR(100),
  access_level_description TEXT
);

CREATE TABLE semester (
  semester_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  academic_cycle VARCHAR(9) NOT NULL,
  semester_term SMALLINT NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  CONSTRAINT chk_semester_dates CHECK (start_date < end_date)
);

CREATE TABLE course (
  course_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  course_code VARCHAR(10) NOT NULL UNIQUE,
  short_name VARCHAR(50) NOT NULL,
  long_name VARCHAR(100),
  course_description TEXT
);

-- =================================================================
-- 2. TABLAS DEPENDIENTES (Con Claves Foráneas)
-- =================================================================

CREATE TABLE app_user (
  user_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  login VARCHAR(50) NOT NULL UNIQUE,
  user_name VARCHAR(100) NOT NULL,
  password VARCHAR(255) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  access_level_id INTEGER NOT NULL,
  creator_user_id INTEGER NOT NULL,
  CONSTRAINT fk_user_access_level FOREIGN KEY (access_level_id) 
    REFERENCES access_level (access_level_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_user_creator FOREIGN KEY (creator_user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE student (
  student_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  ss_number CHAR(11) NOT NULL UNIQUE,
  last_name VARCHAR(50) NOT NULL,
  first_name VARCHAR(50) NOT NULL,
  middle_name VARCHAR(50),
  gender VARCHAR(6) NOT NULL DEFAULT 'MALE',
  dob DATE NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE,
  mobile VARCHAR(15),
  htel VARCHAR(15) NOT NULL,
  wtel VARCHAR(15),
  address_line1 VARCHAR(100) NOT NULL,
  address_line2 VARCHAR(100),
  city VARCHAR(50) NOT NULL,
  state VARCHAR(50) NOT NULL,
  postal_code VARCHAR(10) NOT NULL,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_student_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE lecturer (
  lecturer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  employee_number VARCHAR(20) NOT NULL UNIQUE,
  ss_number CHAR(11) NOT NULL UNIQUE,
  last_name VARCHAR(50) NOT NULL,
  first_name VARCHAR(50) NOT NULL,
  middle_name VARCHAR(50),
  gender VARCHAR(6) NOT NULL DEFAULT 'MALE',
  email VARCHAR(100) NOT NULL UNIQUE,
  mobile VARCHAR(15),
  htel VARCHAR(15),
  wtel VARCHAR(15),
  about TEXT,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_lecturer_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE log_entry (
  log_entry_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  start_datetime TIMESTAMP NOT NULL,
  end_datetime TIMESTAMP,
  table_name VARCHAR(50) NOT NULL,
  table_pk VARCHAR(50) NOT NULL,
  change_type VARCHAR(20) NOT NULL,
  details TEXT,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_log_entry_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT chk_log_times CHECK (end_datetime IS NULL OR start_datetime <= end_datetime)
);

CREATE TABLE scheduled_class (
  class_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  schedule_code VARCHAR(20) NOT NULL UNIQUE,
  section CHAR(1) NOT NULL,
  day VARCHAR(10) NOT NULL,
  time TIME NOT NULL,
  location VARCHAR(75),
  semester_id INTEGER NOT NULL,
  course_id INTEGER NOT NULL,
  lecturer_id INTEGER NOT NULL,
  CONSTRAINT fk_scheduled_class_semester FOREIGN KEY (semester_id) 
    REFERENCES semester (semester_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_scheduled_class_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_scheduled_class_lecturer FOREIGN KEY (lecturer_id) 
    REFERENCES lecturer (lecturer_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

-- =================================================================
-- 3. TABLAS DE UNIÓN (Relaciones Muchos a Muchos)
-- =================================================================

CREATE TABLE course_prerequisite (
  course_prerequisite_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  min_pass_grade VARCHAR(2),
  course_id INTEGER NOT NULL,
  prereq_course_id INTEGER NOT NULL,
  CONSTRAINT fk_prereq_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_prereq_prereq_course FOREIGN KEY (prereq_course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT chk_no_self_prereq CHECK (course_id <> prereq_course_id)
);

CREATE TABLE course_lecturer (
  course_lecturer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  course_id INTEGER NOT NULL,
  lecturer_id INTEGER NOT NULL,
  CONSTRAINT fk_course_lect_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_course_lect_lecturer FOREIGN KEY (lecturer_id) 
    REFERENCES lecturer (lecturer_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT uq_course_lecturer UNIQUE (course_id, lecturer_id)
);

CREATE TABLE student_class (
  student_class_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  registered_date TIMESTAMP NOT NULL,
  student_id INTEGER NOT NULL,
  class_id BIGINT NOT NULL,
  CONSTRAINT fk_student_class_student FOREIGN KEY (student_id)  
    REFERENCES student (student_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_student_class_class FOREIGN KEY (class_id) 
    REFERENCES scheduled_class (class_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT uq_student_class UNIQUE (student_id, class_id)
);

-- =================================================================
-- 4. CREACIÓN DE ÍNDICES PARA OPTIMIZACIÓN (SQL Estándar)
-- =================================================================

CREATE INDEX idx_student_full_name ON student (last_name, first_name);
CREATE INDEX idx_student_dob ON student (dob);
CREATE INDEX idx_student_postal_code ON student (postal_code);
CREATE INDEX idx_lecturer_full_name ON lecturer (last_name, first_name);
CREATE INDEX idx_scheduled_class_code ON scheduled_class (schedule_code);
CREATE INDEX idx_student_class_reg_date ON student_class (registered_date);
```

##### 6.10 Adaptación e Implementación en PostgreSQL 16+

Si bien el script SQL-92/SQL:2016 es teóricamente portable, cada RDBMS posee particularidades sintácticas y tipos de datos nativos que optimizan el rendimiento. PostgreSQL 16+ es uno de los motores más robustos y adherentes a los estándares. A continuación, se presentan las adaptaciones pertinentes:

1. **Reemplazo de `CLOB` por `TEXT`:** PostgreSQL no utiliza el tipo `CLOB` del estándar ANSI. En su lugar, utiliza `TEXT`, el cual es altamente optimizado, no tiene límite de longitud práctico y maneja el almacenamiento fuera de línea (TOAST) de manera transparente.
2. **Identidad de Columnas:** Se mantiene `GENERATED ALWAYS AS IDENTITY`, ya que es el estándar moderno recomendado por PostgreSQL (a partir de la versión 10) en lugar del antiguo pseudo-tipo `SERIAL`, pues ofrece un mejor cumplimiento de los estándares SQL y un manejo más seguro de los permisos de secuencias.
3. **Simulación del tipo `YEAR`:** PostgreSQL no posee un tipo de dato `YEAR` nativo. Se utiliza `SMALLINT` acompañado de una restricción `CHECK` para garantizar la integridad del dominio (ej. años entre 2000 y 2100).
4. **Plegado de Mayúsculas/Minúsculas:** PostgreSQL convierte automáticamente los identificadores no entrecomillados a minúsculas. Por lo tanto, `course_id` se almacenará internamente como `course_id`. Esto es una característica nativa y no afecta la lógica, pero es fundamental saberlo al consultar las tablas.

**Script de Implementación para PostgreSQL 16+**
```sql
-- =================================================================
-- 1. TABLAS INDEPENDIENTES (Sin Claves Foráneas)
-- =================================================================

CREATE TABLE access_level (
  access_level_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  access_level_code VARCHAR(10) NOT NULL UNIQUE,
  short_name VARCHAR(50) NOT NULL,
  long_name VARCHAR(100),
  access_level_description TEXT
);

CREATE TABLE semester (
  semester_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  academic_cycle VARCHAR(9) NOT NULL,
  semester_term SMALLINT NOT NULL CHECK (semester_term BETWEEN 1 AND 2),
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  CONSTRAINT chk_semester_dates CHECK (start_date < end_date)
);

CREATE TABLE course (
  course_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  course_code VARCHAR(10) NOT NULL UNIQUE,
  short_name VARCHAR(50) NOT NULL,
  long_name VARCHAR(100),
  course_description TEXT
);

-- =================================================================
-- 2. TABLAS DEPENDIENTES (Con Claves Foráneas)
-- =================================================================

CREATE TABLE app_user (
  user_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  login VARCHAR(50) NOT NULL UNIQUE,
  user_name VARCHAR(100) NOT NULL,
  password VARCHAR(255) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  access_level_id INTEGER NOT NULL,
  creator_user_id INTEGER NOT NULL,
  CONSTRAINT fk_user_access_level FOREIGN KEY (access_level_id) 
    REFERENCES access_level (access_level_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_user_creator FOREIGN KEY (creator_user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE student (
  student_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  ss_number CHAR(11) NOT NULL UNIQUE,
  last_name VARCHAR(50) NOT NULL,
  first_name VARCHAR(50) NOT NULL,
  middle_name VARCHAR(50),
  gender VARCHAR(6) NOT NULL DEFAULT 'MALE',
  dob DATE NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE,
  mobile VARCHAR(15),
  htel VARCHAR(15) NOT NULL,
  wtel VARCHAR(15),
  address_line1 VARCHAR(100) NOT NULL,
  address_line2 VARCHAR(100),
  city VARCHAR(50) NOT NULL,
  state VARCHAR(50) NOT NULL,
  postal_code VARCHAR(10) NOT NULL,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_student_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE lecturer (
  lecturer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  employee_number VARCHAR(20) NOT NULL UNIQUE,
  ss_number CHAR(11) NOT NULL UNIQUE,
  last_name VARCHAR(50) NOT NULL,
  first_name VARCHAR(50) NOT NULL,
  middle_name VARCHAR(50),
  gender VARCHAR(6) NOT NULL DEFAULT 'MALE',
  email VARCHAR(100) NOT NULL UNIQUE,
  mobile VARCHAR(15),
  htel VARCHAR(15),
  wtel VARCHAR(15),
  about TEXT,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_lecturer_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE log_entry (
  log_entry_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  start_datetime TIMESTAMP NOT NULL,
  end_datetime TIMESTAMP,
  table_name VARCHAR(50) NOT NULL,
  table_pk VARCHAR(50) NOT NULL,
  change_type VARCHAR(20) NOT NULL CHECK (change_type IN ('insercion', 'actualizacion', 'borrado')),
  details TEXT,
  user_id INTEGER NOT NULL,
  CONSTRAINT fk_log_entry_user FOREIGN KEY (user_id) 
    REFERENCES app_user (user_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT chk_log_times CHECK (end_datetime IS NULL OR start_datetime <= end_datetime)
);

CREATE TABLE scheduled_class (
  class_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  schedule_code VARCHAR(20) NOT NULL UNIQUE,
  section CHAR(1) NOT NULL,
  day VARCHAR(10) NOT NULL,
  time TIME NOT NULL,
  location VARCHAR(75),
  semester_id INTEGER NOT NULL,
  course_id INTEGER NOT NULL,
  lecturer_id INTEGER NOT NULL,
  CONSTRAINT fk_scheduled_class_semester FOREIGN KEY (semester_id) 
    REFERENCES semester (semester_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_scheduled_class_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_scheduled_class_lecturer FOREIGN KEY (lecturer_id) 
    REFERENCES lecturer (lecturer_id) ON UPDATE CASCADE ON DELETE RESTRICT
);

-- =================================================================
-- 3. TABLAS DE UNIÓN (Relaciones Muchos a Muchos)
-- =================================================================

CREATE TABLE course_prerequisite (
  course_prerequisite_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  min_pass_grade VARCHAR(2),
  course_id INTEGER NOT NULL,
  prereq_course_id INTEGER NOT NULL,
  CONSTRAINT fk_prereq_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_prereq_prereq_course FOREIGN KEY (prereq_course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT chk_no_self_prereq CHECK (course_id <> prereq_course_id)
);

CREATE TABLE course_lecturer (
  course_lecturer_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  course_id INTEGER NOT NULL,
  lecturer_id INTEGER NOT NULL,
  CONSTRAINT fk_course_lect_course FOREIGN KEY (course_id) 
    REFERENCES course (course_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_course_lect_lecturer FOREIGN KEY (lecturer_id) 
    REFERENCES lecturer (lecturer_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT uq_course_lecturer UNIQUE (course_id, lecturer_id)
);

CREATE TABLE student_class (
  student_class_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  registered_date TIMESTAMP NOT NULL,
  student_id INTEGER NOT NULL,
  class_id BIGINT NOT NULL,
  CONSTRAINT fk_student_class_student FOREIGN KEY (student_id)  
    REFERENCES student (student_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT fk_student_class_class FOREIGN KEY (class_id) 
    REFERENCES scheduled_class (class_id) ON UPDATE CASCADE ON DELETE RESTRICT,
  CONSTRAINT uq_student_class UNIQUE (student_id, class_id)
);

-- =================================================================
-- 4. CREACIÓN DE ÍNDICES PARA OPTIMIZACIÓN (PostgreSQL)
-- =================================================================

CREATE INDEX idx_student_full_name ON student (last_name, first_name);
CREATE INDEX idx_student_dob ON student (dob);
CREATE INDEX idx_student_postal_code ON student (postal_code);
CREATE INDEX idx_lecturer_full_name ON lecturer (last_name, first_name);
CREATE INDEX idx_scheduled_class_code ON scheduled_class (schedule_code);
CREATE INDEX idx_student_class_reg_date ON student_class (registered_date);

-- Nota para PostgreSQL: El motor crea automáticamente índices B-Tree 
-- en todas las columnas que forman parte de una restricción FOREIGN KEY 
-- y UNIQUE, por lo que no es necesario declararlos manualmente para las FKs.
```

**Ventajas de esta implementación en PostgreSQL 16+**

-   **Rendimiento en Texto:** El uso de `TEXT` en lugar de `VARCHAR(n)` para campos como `course_description` o `about` elimina la sobrecarga de validación de longitud en el motor, ya que PostgreSQL maneja ambos de manera idéntica en cuanto a rendimiento, pero `TEXT` es más flexible.  
-   **Integridad de Fechas:** La restricción `CHECK (end_datetime IS NULL OR start_datetime <= end_datetime)` es evaluada de manera extremadamente eficiente por el motor de PostgreSQL, previniendo datos lógicamente imposibles a nivel de base de datos, no solo a nivel de aplicación.  
-   **Concurrencia:** El uso de `GENERATED ALWAYS AS IDENTITY` se apoya en objetos `SEQUENCE` nativos de PostgreSQL, los cuales son seguros para transacciones concurrentes (MVCC), evitando bloqueos o condiciones de carrera al insertar múltiples registros simultáneamente.  

---

#### Conclusión

Con la adaptación a PostgreSQL 16+ y la estandarización rigurosa de la nomenclatura en `snake_case` singular, el ciclo de diseño alcanza su máxima madurez. Hemos transitado desde las tablas planas y redundantes de la "ceguera" inicial del cliente, pasando por el rigor de la Matriz Entidad-Entidad y las afirmaciones de obligatoriedad, hasta transformar un requerimiento narrativo en un modelo relacional **normalizado hasta la 3FN**. 

El resultado no es solo un conjunto de tablas, sino una **arquitectura de datos resiliente**, capaz de soportar el crecimiento de la institución, garantizar la trazabilidad de sus operaciones y servir como cimiento confiable para cualquier capa de aplicación que se construya sobre ella.

> **Recuerda:** Tenga siempre a mano el Manual de Referencia del RDBMS al implementar su diseño, ya que las particularidades de los tipos de datos y los motores de almacenamiento pueden requerir ajustes menores en la sintaxis de los índices y las restricciones. (Captain, 2015, p. 200).

#### Referencias

- Captain, F. A. (2015). *Six-step relational database design: A step by step approach to relational database design and development*. Fidel Captain.  
- Everest, G. C. (1976). Basic data structure models explained with a common example. In *Computing Systems 1976, Proceedings Fifth Texas Conference on Computing Systems* (pp. 39-46). IEEE Computer Society Publications Office.  
- Digital Equipment Corporation. (1993) ANSI X3.135-1992, Database Language SQL. Maynard, Massachusetts <https://web.cecs.pdx.edu/~len/sql-92.pdf>  
