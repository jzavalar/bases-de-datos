## Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas  
## Parte 1: Marco Metodológico  

**Universidad Autónoma Metropolitana, Unidad Iztapalapa**  
**Licenciatura:** Computación  
**UEAs:** Bases de Datos (2151106), Análisis y Diseño de Sistemas () e Ingeniería de Software ()  
**Autor:** Dr. Jesús Zavala Ruiz  
**Última Actualización:** 22 de mayo de 2026  

---

### 1.1. Contexto

El presente documento establece el marco metodológico que guiará la construcción integral del estudio de caso *Pagila*, diseñado como instrumento pedagógico para la UEA Bases de Datos (2151106) en la Licenciatura en Computación de la Universidad Autónoma Metropolitana, Unidad Iztapalapa. El estudio se fundamenta en un modelo de negocio ficticio de carácter histórico: un establecimiento físico tradicional de renta de contenidos audiovisuales, característico de las décadas de 1980 y 1990, en el cual los clientes acudían presencialmente a una tienda y retiraban una copia física de una película en formato DVD o VHS, bajo un modelo operativo análogo al de cadenas como Blockbuster. Si bien este escenario ha sido desplazado por las plataformas de distribución digital contemporáneas, conserva una estructura excepcionalmente valiosa para la formación en ingeniería de datos, ya que encapsula patrones relacionales complejos, restricciones de negocio explícitas, flujos transaccionales presenciales, control de inventario físico y una jerarquía geográfica normalizada.

El artefacto técnico de referencia es la base de datos `Sakila`, originalmente desarrollada por la comunidad de MySQL como esquema de demostración, evaluación de rendimiento y material didáctico. Posteriormente, fue adaptada, portada y renombrada como `pagila` para su ejecución nativa en PostgreSQL, aprovechando las capacidades avanzadas de este motor relacional en materia de integridad, tipos de datos, particionamiento y extensibilidad. Este estudio de caso no persigue la replicación mecánica del script, sino la reconstrucción rigurosa, con fines académicos, de los antecedentes analíticos y de diseño que dieron origen a dicha estructura, demostrando que todo esquema relacional es la materialización técnica de un dominio de negocio previamente formalizado.

### 1.2. Objetivo del Estudio de Caso

El propósito central del ejercicio es construir un estudio de caso técnicamente completo que permita comprender, documentar y validar el proceso integral (*end-to-end*) de análisis y diseño de sistemas de información. A través de este caso, se demostrará cómo los requisitos operativos, las reglas de negocio y la estructura organizacional de un dominio físico se traducen progresivamente en modelos conceptuales, diseños lógicos independientes de tecnología y, finalmente, en un esquema físico optimizado para un sistema gestor de bases de datos (SGBD) específico. El estudio busca cerrar la brecha pedagógica entre la ejecución sintáctica de sentencias DDL/DML y la comprensión profunda de los fundamentos estructurales, semánticos y metodológicos que sustentan la arquitectura de sistemas de información.

### 1.3. Metodología

La estrategia académica adoptada se estructura en dos fases interconectadas que conforman un circuito cerrado de trazabilidad bidireccional:

1.  **Ciclo de Ingeniería Inversa Progresiva:** Partiendo del script `pagila-schema.sql` como punto de partida material, se asciende analíticamente capa por capa para deconstruir la implementación física y reconstruir progresivamente el diseño lógico, la especificación de requisitos, el dominio semántico y, finalmente, el plan operativo del negocio ficticio.
2.  **Ciclo de Análisis y Diseño de Sistemas (Ingeniería Directa de Validación):** Una vez conceptualizado el negocio, se inicia un descenso estructurado que parte del modelo operativo, formaliza el vocabulario y las reglas de negocio, deriva los requisitos de sistema, construye el diseño lógico independiente de tecnología y culmina en la especificación del diseño físico, cerrando el ciclo al validar su correspondencia estructural con el esquema `pagila` original.

Este enfoque bidireccional garantiza que cada artefacto académico no constituya un documento aislado, sino un eslabón verificable dentro de una cadena de ingeniería de sistemas formal y documentada.

#### 1.3.1. Ciclo de Ingeniería Inversa Progresiva

Este ciclo corresponde a la fase de **deconstrucción analítica**. Su finalidad es extraer, a partir del código fuente del esquema, las decisiones implícitas de diseño, las restricciones de negocio codificadas en el DDL, los patrones de normalización aplicados y la arquitectura de datos subyacente. Cada paso representa un nivel de abstracción creciente, transitando desde la sintaxis dependiente del SGBD hacia la semántica independiente del negocio. El objetivo es responder a la pregunta: *¿Qué modelo operativo y qué reglas de negocio se encuentran cristalizadas en esta estructura relacional?*

| Fase | Nivel de Abstracción | Actividad Central | Producto Académico Generado |
|------|---------------------|-------------------|-----------------------------|
| **1. Esquema Físico** | Implementación materializada (PostgreSQL) | Análisis del DDL, tipos nativos, restricciones, particionamiento, disparadores y vistas. | Script `pagila-schema.sql` de referencia |
| **2. Diseño Físico Documentado** | Dependiente del SGBD | Formalización de la materialización: tipos, índices, estrategias de partición, parámetros de almacenamiento y objetos procedimentales. | Documento de Diseño Físico |
| **3. Diseño Lógico** | Independiente de tecnología | Abstracción del modelo relacional, verificación de formas normales (1FN–3FN), definición de entidades, atributos, PK/FK, cardinalidades y vistas conceptuales. | Documento de Diseño Lógico |
| **4. Especificación de Requisitos** | Funcional / Normativa | Inferencia de necesidades del negocio a partir de la estructura lógica. Redacción de RF, RD, RNF con criterios de verificación y trazabilidad (IEEE/ISO). | Especificación de Requisitos (SRS) |
| **5. Dominio Semántico** | Conceptual / Lingüístico | Reinterpretación formal del vocabulario operativo. Desambiguación de términos, explicitación de reglas implícitas y definición de restricciones abstractas. | Documento de Vocabulario y Reinterpretación Semántica |
| **6. Conceptualización del Negocio** | Estratégico / Operativo | Síntesis del modelo operativo ficticio: procesos nucleares, actores, activos físicos (DVD/VHS), reglas de renta presencial y marco de cumplimiento. | Documento Conceptual del Negocio |

#### 1.3.2. Ciclo de Análisis y Diseño de Sistemas

Este ciclo corresponde a la fase de **reconstrucción y validación prospectiva**. Su propósito es demostrar que el conocimiento extraído en la ingeniería inversa es suficiente, coherente y formalmente operativo. Partiendo del plan de negocio reconstruido, se desciende aplicando métodos estándar de ingeniería de software y bases de datos, generando artefactos normalizados que deben poder regenerar el esquema original sin pérdida semántica ni desviación técnica. El objetivo es responder a la pregunta: *¿Es posible reconstruir el esquema inicial partiendo exclusivamente de la especificación conceptual y los requisitos inferidos?*

| Paso | Transición | Mecanismo de Validación |
|------|------------|-------------------------|
| **7** | Plan Operativo → Dominio | Verificación de que cada proceso, actor y restricción estratégica se refleja sin ambigüedad en el vocabulario formalizado. |
| **8** | Dominio → Requisitos | Confirmación de que cada regla semántica se traduce en requisitos verificables, trazables y alineados con estándares de ingeniería de software. |
| **9** | Requisitos → Diseño Lógico | Materialización de los requisitos en un modelo Entidad-Relación formal, garantizando integridad estructural, normalización y patrones de mapeo relacional. |
| **10** | Diseño Lógico → Diseño Físico | Traducción del modelo lógico a objetos PostgreSQL, justificando la selección de tipos de datos, estrategias de indexación, particionamiento y mecanismos de auditoría. |
| **11** | Diseño Físico → Esquema Original | Cotejo estructural exhaustivo contra `pagila-schema.sql` para demostrar que el ciclo cierra sin distorsiones semánticas ni desviaciones técnicas. |

El cierre exitoso de este ciclo descendente constituye la **prueba de redondez metodológica**, demostrando que el proceso de reconstrucción no introdujo artefactos espurios y que el esquema original es la consecuencia lógica, verificable y reproducible del modelo de negocio inferido.

### 1.4. Estado Actual

En este momento, se ha completado el ciclo de ingeniería inversa progresiva y se da inicio formal a la **fase inaugural del ciclo de análisis y diseño del sistema**. Se ha concluido la extracción y documentación del esquema `pagila`, y se han generado las primeras versiones de los artefactos correspondientes a la ruta ascendente: diseño físico documentado, diseño lógico independiente de tecnología, especificación de requisitos, vocabulario de negocio y modelo operativo del negocio ficticio.

El siguiente paso metodológico consiste en ejecutar sistemáticamente el **ciclo descendente de validación**, partiendo del modelo de negocio reconstruido para generar, con rigor trazable, los documentos de requisitos, diseño lógico y diseño físico que culminen en la regeneración conceptual del esquema original. Cada fase se documentará conforme a criterios de verificabilidad, independencia tecnológica (en los niveles lógicos y conceptuales) y correspondencia estricta con los estándares académicos de la UAM.

### 1.5. Fundamento Pedagógico y Competencias a Desarrollar

Esta metodología trasciende la mera ejecución técnica de scripts SQL. Su aplicación en el contexto académico permite a los estudiantes desarrollar las siguientes competencias disciplinares y transversales:

- **Comprender la causalidad estructural:** Una base de datos no es un repositorio aislado, sino la cristalización técnica de un modelo de negocio con reglas, actores y restricciones operativas formalmente definidas.
- **Desarrollar pensamiento abstracto-constructivo:** Capacidad para transitar entre niveles de abstracción (físico → lógico → conceptual → operativo → requisitos → físico) sin pérdida de coherencia semántica ni trazabilidad.
- **Aplicar ingeniería de requisitos formal:** Redacción, verificación y gestión de trazabilidad de especificaciones conforme a estándares IEEE/ISO/INCOSE en un entorno controlado, reproducible y pedagógicamente acotado.
- **Ejercitar auditoría estructural:** Validar esquemas existentes mediante análisis inverso, inspección documental y pruebas de integridad, cultivando el rigor propio de la ingeniería de sistemas profesional.
- **Integrar análisis y diseño:** Cerrar la brecha entre la conceptualización del dominio y la materialización técnica, demostrando que el diseño de datos es un proceso iterativo, documentable y rigurosamente validable.

### 1.6. Conclusión Preliminar

El estudio de caso *Pagila* constituye un ejercicio integral de ingeniería de sistemas, diseñado para demostrar que el análisis y diseño de bases de datos es un proceso cíclico, formal y fundamentado en principios de abstracción, normalización y trazabilidad. Al reconstruir el camino inverso desde la implementación física hasta la conceptualización del negocio, y al descender nuevamente mediante un ciclo de análisis y diseño validado, se garantiza una correspondencia verificable entre los fundamentos teóricos y la arquitectura material del sistema de información.

Este marco metodológico guiará el desarrollo sucesivo de las partes siguientes del estudio de caso, asegurando coherencia estructural, rigor académico y excelencia pedagógica en la formación de profesionales capaces de diseñar, criticar, auditar y validar sistemas de información con solvencia técnica y metodológica.

---

### 1.7. Descargo de Responsabilidad (Disclaimer)

**Aviso de Carácter Académico y Limitaciones de Uso**

1.  **Propósito Exclusivamente Educativo:** El presente documento y los artefactos derivados del Estudio de Caso *Pagila* han sido elaborados con fines estrictamente pedagógicos para la Unidad de Enseñanza-Aprendizaje Bases de Datos (2151106) de la Licenciatura en Computación, Universidad Autónoma Metropolitana, Unidad Iztapalapa. Su contenido no constituye asesoría profesional, especificación contractual ni guía para implementación en entornos productivos.

2.  **Naturaleza Ficticia del Modelo de Negocio:** El negocio de renta de películas descrito en este estudio de caso es una reconstrucción hipotética con fines ilustrativos. Cualquier similitud con empresas reales, existentes o extintas (incluyendo, sin limitación, Blockbuster, Video Club, o cualquier otra cadena de renta de contenido audiovisual), es mera coincidencia y no implica respaldo, afiliación, validación o relación alguna con dichas organizaciones.

3.  **Reconstrucción Hipotética de Antecedentes:** Los documentos de requisitos, diseño lógico, vocabulario de negocio y plan operativo generados mediante la metodología de Ingeniería Inversa Progresiva representan una interpretación académica plausible de los antecedentes que pudieron originar el esquema `pagila`. No pretenden reflejar con exactitud histórica las decisiones reales tomadas por los desarrolladores originales de `Sakila` o `pagila`, ni constituyen documentación oficial de dichos proyectos.

4.  **Independencia de Proveedores Tecnológicos:** Las referencias a MySQL, PostgreSQL, o cualquier otro sistema gestor de bases de datos, lenguaje de programación o herramienta de desarrollo se realizan con fines ilustrativos y pedagógicos. Este documento no está patrocinado, avalado ni afiliado a Oracle Corporation, The PostgreSQL Global Development Group, ni a ninguna otra entidad comercial mencionada.

5.  **Ausencia de Garantías:** Los artefactos, modelos, especificaciones y scripts derivados de este estudio de caso se proporcionan "tal cual" (*as is*), sin garantía expresa o implícita de exactitud, integridad, idoneidad para un propósito particular, no violación de derechos de terceros o disponibilidad continua. La Universidad Autónoma Metropolitana, sus docentes y colaboradores no asumen responsabilidad por errores, omisiones, daños directos, indirectos o consecuentes derivados del uso o interpretación de este material.

6.  **Propiedad Intelectual y Atribución:** 
    - El esquema original `Sakila` es un proyecto de código abierto desarrollado por la comunidad de MySQL. 
    - La adaptación `Pagila` para PostgreSQL es un esfuerzo comunitario disponible en repositorios públicos. 
    - Los documentos metodológicos, de análisis y diseño elaborados en el marco de este estudio de caso son propiedad académica de la UAM-Iztapalapa y se distribuyen bajo licencia Creative Commons Atribución-NoComercial-CompartirIgual 4.0 Internacional (CC BY-NC-SA 4.0), salvo indicación expresa en contrario.
    - Se requiere citación completa de la fuente para cualquier reproducción, adaptación o uso académico: *Zavala Ruiz, J. (2026). Estudio de Caso Pagila: Análisis y Diseño del Sistema de Información para un Negocio Ficticio de Renta de Películas. Universidad Autónoma Metropolitana, Unidad Iztapalapa.*

7.  **Limitación de Responsabilidad Profesional:** Este material no sustituye la formación profesional certificada en ingeniería de software, arquitectura de datos o administración de bases de datos. Los estudiantes y docentes que utilicen este estudio de caso asumen la responsabilidad de validar, adaptar y contextualizar sus contenidos conforme a los requisitos específicos de sus proyectos, entornos institucionales y marcos normativos aplicables.

8.  **Actualización y Vigencia:** Los contenidos de este documento reflejan el estado del conocimiento y las prácticas pedagógicas a la fecha de última actualización indicada. La evolución tecnológica, normativa y metodológica puede requerir revisiones posteriores. Se recomienda consultar fuentes primarias y documentación oficial para decisiones de implementación técnica.

> **Nota Final:** 
> Este estudio de caso forma parte de un ejercicio de aprendizaje reflexivo en ingeniería de software. Su valor reside en el proceso de análisis, crítica y reconstrucción metodológica, no en la reproducción literal de artefactos. Se invita a los usuarios a cuestionar, validar y mejorar las propuestas aquí presentadas, en espíritu de rigor académico y desarrollo profesional responsable.

---

**Universidad Autónoma Metropolitana - Unidad Iztapalapa**  
*Licenciatura en Computación | División de Ciencias Básicas e Ingeniería*  
*Documento de distribución educativa bajo licencia CC BY-NC-SA 4.0*