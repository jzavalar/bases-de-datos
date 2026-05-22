# Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas  
## Parte 3: Especificación de Requerimientos de Software  

**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Licenciatura:** Computación  
**UEAs:** Bases de Datos (2151106), Análisis y Diseño de Sistemas, Ingeniería de Software  
**Autor:** Dr. Jesús Zavala Ruiz  
**Última Actualización:** 22 de mayo de 2026  

---

### Introducción: La Ingeniería de Requerimientos como Fundamento del Diseño

Antes de adentrarnos en la especificación técnica, conviene detenernos brevemente en el propósito y la naturaleza de los requerimientos de software. En términos sencillos, un requerimiento es una condición o capacidad que un sistema debe poseer para satisfacer una necesidad del negocio o resolver un problema identificado. No se trata de describir cómo se construirá el sistema, sino de definir *qué* debe hacer, *para quién* y *bajo qué condiciones*.

La ingeniería de requerimientos es la disciplina que se ocupa de levantar (*to elicit*), analizar, documentar, validar y gestionar estos requisitos a lo largo del ciclo de vida del desarrollo. Su importancia radica en que constituye el puente entre el mundo del negocio y el mundo técnico: sin requerimientos claros, verificables y trazables, incluso el diseño más elegante puede fracasar al no responder a las necesidades reales de sus usuarios.

Sin embargo, esta tarea no está exenta de desafíos. Los requerimientos suelen ser incompletos al inicio, ambiguos en su redacción, contradictorios entre sí o sujetos a cambios frecuentes. Por ello, estándares internacionales como IEEE 830-1993 e ISO/IEC/IEEE 29148:2018 proponen características de calidad para los requisitos: deben ser correctos, no ambiguos, completos, consistentes, clasificables por prioridad, verificables, modificables y trazables. Cumplir con estos criterios no garantiza el éxito del proyecto, pero reduce significativamente el riesgo de malentendidos costosos.

En el contexto académico de este estudio de caso, la especificación que presentamos a continuación no busca ser exhaustiva en el sentido industrial, sino demostrar cómo los conceptos, reglas y procesos descritos en la Parte 2 se transforman sistemáticamente en requisitos formales, listos para guiar el diseño lógico y físico del sistema. Cada requisito incluye un identificador único, una descripción clara, criterios de verificación y trazabilidad hacia el vocabulario de negocio del Anexo A, garantizando que ninguna decisión de diseño quede desconectada de una necesidad operativa real.

### 3.1. Propósito y Alcance de la Especificación

El presente documento formaliza los requerimientos de software del sistema de información que soporta la operación del negocio ficticio de renta de películas descrito en la Parte 2. Su propósito es establecer, con precisión y verificabilidad, las capacidades funcionales, las restricciones de datos y los atributos de calidad que el sistema debe cumplir para habilitar los procesos operativos, garantizar la integridad de la información y satisfacer las expectativas de los actores involucrados.

Este artefacto constituye el eslabón central entre la conceptualización del negocio y el diseño técnico. A partir de estos requerimientos, se derivará el modelo lógico independiente de tecnología y, posteriormente, el esquema físico implementado en PostgreSQL, cerrando el ciclo de ingeniería inversa progresiva al validar que la materialización técnica corresponde con las necesidades originales del dominio.

**Lo que incluye esta especificación:**
- Requisitos funcionales organizados por proceso operativo, con criterios de aceptación verificables.
- Requisitos de datos que definen la estructura, integridad y calidad de la información gestionada.
- Requisitos no funcionales que establecen condiciones de desempeño, disponibilidad, seguridad y escalabilidad.
- Matriz de trazabilidad que vincula cada requisito con los conceptos y reglas del negocio descritos en la Parte 2 y su Anexo A.

**Lo que queda fuera:**
- Diseño de interfaces de usuario, flujos de interacción o prototipos visuales.
- Especificación de arquitecturas técnicas, lenguajes de programación o plataformas de despliegue.
- Planificación de proyecto, estimación de esfuerzos o cronogramas de implementación.

### 3.2. Descripción General del Producto

#### 3.2.1. Perspectiva del Sistema

El sistema de información que se especifica opera como el núcleo de soporte a decisiones y transacciones de una cadena de establecimientos de renta de contenido audiovisual en formato físico. No es una aplicación independiente, sino un componente central que interactúa con usuarios operativos (empleados de mostrador, administradores) y proporciona datos para reportes analíticos. Su arquitectura de referencia sigue un modelo cliente-servidor, con separación lógica entre capa de presentación, lógica de negocio y gestión de datos, aunque esta especificación se mantiene independiente de decisiones de implementación tecnológica.

#### 3.2.2. Funciones Principales del Producto

El sistema debe habilitar las siguientes capacidades operativas:

- Mantener un catálogo maestro de títulos audiovisuales con metadatos descriptivos, clasificación etaria, tarifas y soporte multilingüe.
- Gestionar la asociación de títulos con participantes del elenco y categorías temáticas, garantizando unicidad en los vínculos.
- Controlar el inventario físico de ejemplares por establecimiento, registrando su ubicación y estado de disponibilidad en tiempo real.
- Procesar transacciones de renta y devolución, vinculando ejemplar, cliente, empleado y marcas temporales, con prevención de asignaciones concurrentes.
- Registrar pagos asociados a rentas, con validación de montos positivos y preservación del historial financiero.
- Normalizar información geográfica (país, ciudad, dirección) para evitar redundancia y facilitar mantenimiento centralizado.
- Generar reportes operativos y analíticos agregados por período, categoría y ubicación para soporte a la toma de decisiones.

#### 3.2.3. Clases de Usuarios y Características

El sistema distingue tres perfiles o roles principales de interacción, cada uno con responsabilidades y necesidades de información diferenciadas:

El **empleado operativo** procesa transacciones cotidianas en mostrador: registra rentas, cobros y devoluciones; consulta disponibilidad de inventario; y da de alta clientes. Requiere credenciales de acceso únicas, interfaz ágil para captura de datos y validaciones en tiempo real que prevengan errores operativos.

El **administrador de tienda** supervisa la operación de una sede: valida reportes, asigna turnos, resuelve incidencias comerciales y gestiona inventario local. Necesita permisos diferenciados, acceso a métricas de desempeño por sede y capacidades de auditoría sobre transacciones procesadas por su equipo.

El **gestor de catálogo** (rol implícito, accesible mediante interfaz administrativa) mantiene la información maestra: da de alta títulos, participantes, categorías e idiomas; negocia tarifas con productoras; y asegura la coherencia transversal del catálogo. Requiere funciones de administración centralizada sin exposición a transacciones operativas.

#### 3.2.4. Entorno de Operación

El sistema opera en un contexto organizacional distribuido: múltiples establecimientos físicos con conectividad hacia un núcleo centralizado de gestión de datos. Las transacciones se procesan en tiempo real en cada sede, con requisitos de disponibilidad continua (24×7×365) y tolerancia a fallos parciales que no comprometan la integridad de operaciones en curso. La soberanía digital exige que los datos personales y transaccionales residan bajo jurisdicción nacional o acuerdos explícitos de gobernanza.

#### 3.2.5. Restricciones de Diseño e Implementación

- El sistema debe mantener independencia conceptual respecto a tecnologías específicas en las fases de requisitos y diseño lógico.
- La estructura de datos debe cumplir con principios de normalización relacional (al menos Tercera Forma Normal) para eliminar redundancias y anomalías.
- Los identificadores de entidades deben ser invariantes y generados por el sistema, no derivados de atributos descriptivos susceptibles de cambio.
- La desactivación lógica de clientes o empleados no debe eliminar ni invalidar historiales transaccionales asociados.

#### 3.2.6. Supuestos y Dependencias

- Se asume que los usuarios operativos cuentan con capacitación básica en el uso del sistema y en los procesos de negocio que soporta.
- Se asume disponibilidad de infraestructura de red que permita comunicación entre sedes y el núcleo central con latencia aceptable para operaciones en tiempo real.
- El sistema depende de la existencia de catálogos maestros inicializados (idiomas, categorías, clasificación etaria) antes del inicio de operaciones transaccionales.

### 3.3. Requerimientos Específicos

#### 3.3.1. Requerimientos Funcionales

Los siguientes requisitos describen las capacidades que el sistema debe exhibir para soportar los procesos operativos del negocio. Cada uno incluye un identificador único, una descripción clara y criterios de verificación reproducibles en entorno de laboratorio.

**RF-01: Gestión del Catálogo Maestro de Títulos**  
El sistema debe permitir registrar, consultar y actualizar títulos audiovisuales con información descriptiva (nombre, sinopsis, año de estreno, duración), clasificación etaria normativa, tarifas comerciales y soporte para múltiples idiomas (versión comercial y, opcionalmente, idioma original).  
*Criterio de verificación:* Inserción de un título con todos los campos obligatorios debe completarse exitosamente; intento de registro con clasificación etaria fuera del catálogo autorizado debe ser rechazado.  
*Trazabilidad:* Vinculado a los términos "Título" y "Clasificación Etaria" del Anexo A.

**RF-02: Asociación de Títulos con Participantes y Categorías**  
El sistema debe permitir asociar un título con múltiples participantes del elenco y múltiples categorías temáticas, garantizando que la combinación (título, participante) y (título, categoría) sea única en cada caso.  
*Criterio de verificación:* Intento de duplicar una asociación existente debe generar error de integridad; consulta por título debe retornar lista completa de participantes y categorías asociadas.  
*Trazabilidad:* Vinculado a los patrones "Asociación Muchos-a-Muchos" y términos "Participante", "Categoría Temática" del Anexo A.

**RF-03: Control de Inventario Físico por Establecimiento**  
El sistema debe asignar un identificador único a cada ejemplar físico, vincularlo explícitamente a un establecimiento y permitir consultar en tiempo real su estado de disponibilidad (disponible, rentado, retirado).  
*Criterio de verificación:* Consulta por título y sede debe retornar conteo de ejemplares totales y disponibles; intento de registrar dos rentas simultáneas para el mismo ejemplar debe ser rechazado.  
*Trazabilidad:* Vinculado al término "Ejemplar / Copia Física" y regla "Disponibilidad Exclusiva por Ejemplar" del Anexo A.

**RF-04: Procesamiento de Transacciones de Renta y Devolución**  
El sistema debe registrar cada renta con fecha y hora de inicio, ejemplar, cliente y empleado procesador; permitir registrar opcionalmente fecha de devolución; y preservar el historial completo incluso ante cambios de estado de clientes o empleados.  
*Criterio de verificación:* Inserción de renta con todos los campos obligatorios debe completarse; actualización de fecha de devolución debe modificar estado de disponibilidad del ejemplar; consulta de historial por cliente debe retornar transacciones históricas incluso si el cliente está desactivado.  
*Trazabilidad:* Vinculado a los términos "Renta", "Devolución" y regla "Preservación del Historial Transaccional" del Anexo A.

**RF-05: Registro y Validación de Pagos**  
El sistema debe vincular obligatoriamente cada pago a una renta válida, validar que los montos sean estrictamente positivos y registrar fecha exacta de procesamiento con empleado responsable.  
*Criterio de verificación:* Intento de registrar pago sin referencia a renta válida debe ser rechazado; inserción de pago con monto negativo debe generar error de dominio; reporte financiero por período debe agregar montos correctamente.  
*Trazabilidad:* Vinculado al término "Pago / Cobro" y regla "Vinculación Financiera Obligatoria" del Anexo A.

**RF-06: Normalización de Información Geográfica**  
El sistema debe estructurar direcciones en jerarquía país → ciudad → dirección, permitiendo que un mismo registro de dirección sea compartido por múltiples actores sin duplicación, e impidiendo la eliminación de niveles superiores mientras existan dependencias activas.  
*Criterio de verificación:* Cambio en nombre de ciudad debe reflejarse automáticamente en todas las direcciones que la referencian; intento de eliminar país con ciudades registradas debe ser rechazado.  
*Trazabilidad:* Vinculado al término "Dirección / Ubicación Física" y patrón "Jerarquía Geográfica" del Anexo A.

**RF-07: Generación de Reportes Operativos y Analíticos**  
El sistema debe permitir consultar disponibilidad de inventario por título y sede, historial de rentas por cliente, directorio de personal por sede, ingresos agregados por categoría y período, y tendencias de popularidad de títulos o participantes.  
*Criterio de verificación:* Ejecución de consulta de disponibilidad debe completarse en ≤ 2 segundos bajo carga nominal; reporte de ingresos por categoría debe retornar agregación correcta de montos de pago.  
*Trazabilidad:* Vinculado a las secciones 2.7.1 y 2.7.2 de la Parte 2.

#### 3.3.2. Requerimientos de Datos

Estos requisitos definen las propiedades estructurales y de calidad que debe poseer la información gestionada por el sistema.

**RD-01: Integridad Referencial Estricta**  
Toda referencia entre registros (por ejemplo, un pago que refiere una renta, una dirección que refiere una ciudad) debe validar la existencia del registro origen antes de permitir la creación del dependiente. La eliminación de un registro maestro debe estar restringida si existen dependencias activas.  
*Criterio de verificación:* Intento de eliminar categoría con títulos clasificados debe generar error; inserción de pago con rental_id inexistente debe ser rechazada.

**RD-02: Unicidad de Identificadores y Asociaciones**  
Cada entidad maestra (título, participante, categoría, ejemplar, cliente, empleado, establecimiento) debe poseer un identificador único e invariante. Las asociaciones muchos-a-muchos (título-participante, título-categoría) deben garantizar unicidad en la combinación de claves.  
*Criterio de verificación:* Inserción duplicada de asociación título-participante debe generar error de clave única.

**RD-03: Validación de Dominios Predefinidos**  
Los atributos con valores restringidos (clasificación etaria, estado operativo) deben validar contra catálogos cerrados; cualquier valor fuera del dominio autorizado debe ser rechazado.  
*Criterio de verificación:* Intento de registrar título con clasificación "X" (no autorizada) debe ser rechazado.

**RD-04: Trazabilidad Temporal Uniforme**  
Todos los registros maestros y transaccionales deben documentar automáticamente el instante de su última modificación mediante una marca temporal actualizada en cada operación de cambio.  
*Criterio de verificación:* Actualización de cualquier campo en un título debe modificar su atributo last_update; consulta de auditoría debe retornar historial de cambios.

**RD-05: Calidad de Información para Toma de Decisiones**  
La información agregada para reportes analíticos debe preservarse con consistencia transversal: cambios en catálogos maestros deben reflejarse automáticamente en vistas dependientes, sin requerir reprocesamiento manual.  
*Criterio de verificación:* Cambio en nombre de categoría debe reflejarse inmediatamente en reporte de ingresos por categoría.

#### 3.3.3. Requerimientos No Funcionales

Estos requisitos establecen atributos de calidad y condiciones de contexto que condicionan el diseño y la operación del sistema.

**RNF-01: Disponibilidad y Tolerancia a Fallos**  
El sistema debe operar de manera continua 24×7×365, tolerando fallos parciales de red o componentes sin comprometer la integridad de transacciones en curso. Las operaciones críticas (renta, pago) deben garantizar atomicidad: o se completan totalmente o no se ejecutan.  
*Criterio de verificación:* Simulación de fallo de red durante transacción de renta debe revertir cambios parciales sin dejar estado inconsistente.

**RNF-02: Desempeño Operativo**  
Las consultas de disponibilidad de inventario y el registro de transacciones de renta deben completarse en ≤ 2 segundos bajo carga concurrente nominal de 50 transacciones simultáneas. Los reportes analíticos agregados deben responder en ≤ 5 segundos para períodos de hasta un año.  
*Criterio de verificación:* Prueba de carga con herramienta estándar debe medir tiempos de respuesta dentro de los límites establecidos.

**RNF-03: Seguridad y Control de Accesos**  
El sistema debe implementar control de accesos basado en roles: empleados operativos solo pueden ejecutar transacciones en su sede asignada; administradores tienen permisos de supervisión ampliados; gestores de catálogo acceden exclusivamente a funciones de administración maestra. Las credenciales de acceso deben almacenarse mediante funciones criptográficas unidireccionales.  
*Criterio de verificación:* Intento de empleado de acceder a funciones administrativas debe ser denegado; consulta a tabla de credenciales no debe retornar contraseñas en claro.

**RNF-04: Escalabilidad y Mantenibilidad**  
La estructura de datos debe permitir incorporar nuevas sedes, expandir catálogo y acumular historial transaccional sin reestructuración de fondo. El diseño debe evitar redundancias mediante normalización y utilizar identificadores estables que faciliten migraciones futuras.  
*Criterio de verificación:* Inserción de nueva sede con inventario inicial no debe requerir modificación de esquema de datos existente.

**RNF-05: Soberanía Digital y Protección de Datos**  
Los datos personales de clientes y empleados, así como los registros transaccionales y de auditoría, deben residir bajo jurisdicción nacional o acuerdos explícitos de gobernanza. El sistema debe soportar técnicas de anonimización controlada para entornos de prueba sin comprometer la integridad de datos productivos.  
*Criterio de verificación:* Procedimiento de anonimización debe enmascarar campos personales (email, teléfono) preservando estructura referencial para pruebas.

#### 3.3.4. Requerimientos de Interfaz Externa

**RIE-01: Interfaz con Usuarios Operativos**  
El sistema debe proporcionar una interfaz de captura ágil para empleados de mostrador, con validaciones en tiempo real que prevengan errores de entrada (por ejemplo, selección de ejemplar no disponible) y mensajes de error claros que guíen la corrección.

**RIE-02: Interfaz para Reportes Analíticos**  
El sistema debe exponer vistas predefinidas o mecanismos de consulta que permitan a administradores y gestores generar reportes agregados sin requerir conocimiento técnico de la estructura subyacente de datos.

### 3.4. Criterios de Verificación y Validación

La calidad de esta especificación se evalúa mediante dos dimensiones complementarias:

**Verificación:** ¿Construimos el producto correctamente? Cada requisito incluye criterios de aceptación reproducibles en entorno de laboratorio, permitiendo demostrar objetivamente su cumplimiento mediante pruebas de inserción, consulta, validación de restricciones y medición de desempeño.

**Validación:** ¿Construimos el producto correcto? La matriz de trazabilidad que se presenta a continuación garantiza que cada requisito derive de una necesidad explícita del negocio descrita en la Parte 2, evitando la inclusión de funcionalidades superfluas o desconectadas del dominio operativo.

### 3.5. Matriz de Trazabilidad: Del Negocio a los Requerimientos

La siguiente tabla vincula los conceptos, reglas y procesos del negocio (Parte 2 y Anexo A) con los requisitos de software que los materializan. Esta trazabilidad bidireccional permite auditar que ninguna decisión de diseño quede desconectada de una necesidad real, y que ningún requisito carezca de justificación operativa.

| Concepto del Negocio (Parte 2 / Anexo A) | Requisito de Software Asociado | Tipo de Requisito |
|-----------------------------------------|--------------------------------|-------------------|
| Título, Clasificación Etaria | RF-01, RD-03 | Funcional, Datos |
| Asociación Muchos-a-Muchos (título-participante, título-categoría) | RF-02, RD-02 | Funcional, Datos |
| Ejemplar / Copia Física, Disponibilidad Exclusiva | RF-03, RNF-02 | Funcional, No Funcional |
| Renta, Devolución, Preservación de Historial | RF-04, RD-01 | Funcional, Datos |
| Pago, Vinculación Financiera Obligatoria | RF-05, RD-01 | Funcional, Datos |
| Dirección, Jerarquía Geográfica | RF-06, RD-01 | Funcional, Datos |
| Reportes Operativos y Analíticos | RF-07, RNF-02 | Funcional, No Funcional |
| Disponibilidad 24×7×365, Tolerancia a Fallos | RNF-01 | No Funcional |
| Control de Accesos por Rol, Seguridad de Credenciales | RNF-03 | No Funcional |
| Escalabilidad, Normalización | RNF-04, RD-01 | No Funcional, Datos |
| Soberanía Digital, Anonimización | RNF-05 | No Funcional |

Esta matriz no es exhaustiva, pero ilustra el principio de trazabilidad que guía todo el estudio de caso: cada elemento técnico debe poder rastrearse hasta una necesidad del negocio, y cada necesidad del negocio debe encontrar expresión verificable en los requisitos del sistema.

### 3.6. Control de Versiones

| Versión | Fecha | Autor | Cambios Principales |
|---------|-------|-------|---------------------|
| 1.0 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Especificación inicial de requerimientos derivada de la conceptualización del negocio (Parte 2) y su Anexo A; alineación con estándares IEEE/ISO. |
| 1.1 | 22-may-2026 | Dr. Jesús Zavala Ruiz | Revisión para mejorar claridad narrativa, reforzar criterios de verificación y consolidar matriz de trazabilidad. |

---

> **Declaración de Propósito Académico:**  
> Este documento ha sido elaborado exclusivamente con fines pedagógicos dentro del estudio de caso de la UEA Bases de Datos (2151106) de la Universidad Autónoma Metropolitana, Unidad Iztapalapa. Su contenido especifica hipotéticamente los requerimientos de software que podrían haber dado origen al esquema `pagila`, integrando principios de ingeniería de requisitos conforme a estándares IEEE/ISO. No constituye un documento contractual ni prescribe tecnologías de implementación específicas.

---

**Elaborado para distribución educativa**  
**Universidad Autónoma Metropolitana - Unidad Iztapalapa**  
*Licenciatura en Computación | UEA Bases de Datos (2151106)*  
*Documento de distribución educativa bajo licencia CC BY-NC-SA 4.0. Se autoriza la reproducción parcial con citación institucional completa.*