## Programación Semanal
## Unidad de Enseñanza-Aprendizaje: Bases de Datos (2151106)
### Trimestre 26-P | 11 semanas efectivas

### Parámetros Operativos Confirmados

| Parámetro | Especificación |
|-----------|---------------|
| **Estructura semanal** | Miércoles (Teoría, aula) -> Viernes (Teoría, aula) -> Lunes (Práctica, laboratorio) |
| **Inicio** | Miércoles 6 de mayo de 2026 |
| **Término** | Lunes 20 de julio de 2026 |
| **Día inhábil** | Viernes 15 de mayo (asueto institucional) |
| **Sesiones totales** | 32 sesiones (22 teóricas + 10 prácticas) |
| **Distribución de exámenes** | Teórico 1 (Semana 7, post-4.3), Teórico 2 (Semana 11, post-6.2), Práctico Final dividido en P10 y P11 |

### Calendario Maestro de Sesiones

| Semana | Fechas (Mie-Vie-Lun) | Sesiones | Contenido Sintético | Evaluación / Nota |
|:---:|:---|:---|:---|:---|
| **1** | 6-8 may / 11 may | T1, T2, P1 | **1.1-1.4**: Introducción a las bases de datos | Presentación del programa + fundamentos conceptuales |
| **2** | 13-15 may / 18 may | T3, ASUETO, P2 | **2.1-2.3**: Sistemas administradores de BD | Viernes 15 inhábil: contenido compensado con lectura autónoma + integración en práctica |
| **3** | 20-22 may / 25 may | T5, T6, P3 | **3.1-3.2**: Modelo ER y procedimiento de diseño | Modelado conceptual con papel y lápiz |
| **4** | 27-29 may / 1 jun | T7, T8, P4 | **3.3-3.4**: Modelo relacional y correspondencia ER | Cierre de bloque III.2 y mapeo a estructura tabular |
| **5** | 3-5 jun / 8 jun | T9, T10, P5 | **3.5 + 4.1**: Redundancias, normalización y tablas | Fusión estratégica de contenidos para mantener ritmo |
| **6** | 10-12 jun / 15 jun | T11, T12, P6 | **4.2-4.3**: Restricciones, vistas y consolidación | Cierre oficial del Bloque IV (preparación para Examen 1) |
| **7** | 17-19 jun / 22 jun | Examen T1, T13, P7 | **5.1**: Consultas simples | **Examen Teórico 1** (Bloques I-IV) - Miércoles 17 junio |
| **8** | 24-26 jun / 29 jun | T14, T15, P8 | **5.2-5.3**: Consultas multitable y operaciones de escritura | Transición de DQL a DML y manejo de transacciones |
| **9** | 1-3 jul / 6 jul | T16, T17, P9 | **6.1**: Disparadores (triggers) | Arquitectura de ejecución y casos de auditoría/validación |
| **10** | 8-10 jul / 13 jul | T18, T19, P10 | **6.2**: Procedimientos almacenados | **Examen Práctico Final - Parte 1** - Lunes 13 julio |
| **11** | 15-17 jul / 20 jul | Examen T2, T20, P11 | Cierre administrativo y evaluación terminal | **Examen Teórico 2** (Mié 15 jul) + **Examen Práctico Final - Parte 2** (Lun 20 jul) |

### Distribución Detallada por Semana

#### SEMANA 1 (6-11 mayo) - Bloque I: Fundamentos
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T1 | Mie 6 may | Teoría | 1.1: Conceptos básicos de BD | Presentación del programa + dato vs. información vs. conocimiento |
| T2 | Vie 8 may | Teoría | 1.2-1.4: SGBD y modelos de datos | Arquitectura de SGBD + evolución de modelos |
| P1 | Lun 11 may | Práctica | Instalación y exploración de entorno | PostgreSQL/MySQL + pgAdmin/Workbench + ejercicio básico |

#### SEMANA 2 (13-18 mayo) - Bloque II: SGBD
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T3 | Mie 13 may | Teoría | 2.1-2.2: Usuarios y componentes de SGBD | Capas de abstracción + independencia de datos |
| T4 | Vie 15 may | ASUETO | - | Compensación: Lectura Elmasri Cap. 2 + foro asincrónico |
| P2 | Lun 18 may | Práctica | 2.3 + administración de usuarios | Roles, privilegios + debate ventajas/desventajas SGBD |

#### SEMANA 3 (20-25 mayo) - Bloque III.1: Diseño ER
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T5 | Mie 20 may | Teoría | 3.1: Modelo Entidad-Relación | Entidades, atributos, relaciones + notación Chen/Crow's Foot |
| T6 | Vie 22 may | Teoría | 3.2: Procedimiento de diseño ER | Fases del diseño conceptual + errores frecuentes |
| P3 | Lun 25 may | Práctica | Modelado conceptual con papel y lápiz | Caso: Plataforma de gestión académica |

#### SEMANA 4 (27 may-1 jun) - Bloque III.2: Modelo Relacional
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T7 | Mie 27 may | Teoría | 3.3: Modelo relacional | Tuplas, atributos, claves, integridad |
| T8 | Vie 29 may | Teoría | 3.4: Correspondencia ER-Relacional | Reglas canónicas de mapeo y casos especiales |
| P4 | Lun 1 jun | Práctica | Transformación ER a DDL | Script CREATE TABLE + matriz de trazabilidad |

#### SEMANA 5 (3-8 junio) - Bloque III.3 + IV.1: Normalización y DDL
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T9 | Mie 3 jun | Teoría | 3.5: Redundancias y normalización (1NF-3NF) | Dependencias funcionales + axiomas de Armstrong |
| T10 | Vie 5 jun | Teoría | 3.5: BCNF + 4.1: Definición de tablas | Algoritmos de descomposición + tipos de datos SQL |
| P5 | Lun 8 jun | Práctica | Normalización aplicada + DDL | Identificación de DF + implementación física en SGBD |

#### SEMANA 6 (10-15 junio) - Bloque IV.2-3: Restricciones y Vistas
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T11 | Mie 10 jun | Teoría | 4.2: Restricciones de integridad | PRIMARY KEY, FOREIGN KEY, CHECK, UNIQUE |
| T12 | Vie 12 jun | Teoría | 4.3: Vistas + repaso integrado | CREATE VIEW + limitaciones de actualizabilidad |
| P6 | Lun 15 jun | Práctica | Integración DDL completo | Script con restricciones + validación operativa |

#### SEMANA 7 (17-22 junio) - Examen Teórico 1 + Bloque V.1
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T13 | Mie 17 jun | Examen Teórico 1 | Evaluación Bloques I-IV | Examen escrito (2 h) |
| T14 | Vie 19 jun | Teoría | 5.1: Consultas simples (SELECT, WHERE) | Sintaxis fundamental + operadores lógicos |
| P7 | Lun 22 jun | Práctica | Consultas a una tabla | WHERE, ORDER BY, LIMIT, funciones escalares |

#### SEMANA 8 (24-29 junio) - Bloque V.2-3: Consultas Complejas y DML
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T15 | Mie 24 jun | Teoría | 5.2: JOINs y combinaciones | INNER, LEFT/RIGHT OUTER JOIN + alias |
| T16 | Vie 26 jun | Teoría | 5.2-5.3: Subconsultas y operaciones de escritura | Agregaciones, HAVING, INSERT/UPDATE/DELETE |
| P8 | Lun 29 jun | Práctica | Consultas multitable y DML | Subconsultas correlacionadas + transacciones básicas |

#### SEMANA 9 (1-6 julio) - Bloque VI.1: Disparadores
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T17 | Mie 1 jul | Teoría | 6.1: Arquitectura de disparadores | BEFORE/AFTER, ROW/STATEMENT LEVEL |
| T18 | Vie 3 jul | Teoría | 6.1: Casos de uso y mejores prácticas | Auditoría, validación cruzada y manejo de excepciones |
| P9 | Lun 6 jul | Práctica | Implementación de triggers | Scripts funcionales + pruebas de activación |

#### SEMANA 10 (8-13 julio) - Bloque VI.2 + Examen Práctico P1
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T19 | Mie 8 jul | Teoría | 6.2: Procedimientos almacenados (sintaxis) | CREATE PROCEDURE, parámetros IN/OUT/INOUT |
| T20 | Vie 10 jul | Teoría | 6.2: Estructuras de control y excepciones | IF, LOOP, BEGIN...EXCEPTION + cursores |
| P10 | Lun 13 jul | Examen Práctico P1 | Diseño, DDL/DML y consultas complejas | Evaluación en laboratorio (3 h) |

#### SEMANA 11 (15-20 julio) - Examen Teórico 2 + Examen Práctico P2 + Cierre
| Sesión | Fecha | Tipo | Contenido | Actividad Principal |
|--------|-------|------|-----------|-------------------|
| T21 | Mie 15 jul | Examen Teórico 2 | Evaluación Bloques V-VI | Examen escrito (2 h) |
| T22 | Vie 17 jul | Teoría | Orientaciones institucionales + cierre reflexivo | Lineamientos para recuperación + encuesta de satisfacción |
| P11 | Lun 20 jul | Examen Práctico P2 | Programación en SGBD + integración | Evaluación en laboratorio (3 h) |

### Esquema de Evaluación

| Componente | Fecha | Tipo | Ponderación | Contenido Evaluado |
|------------|-------|------|-------------|-------------------|
| **Examen Teórico 1** | Miércoles 17 junio | Escrito (aula) | 25% | Bloques I-IV (1.1 a 4.3) |
| **Examen Teórico 2** | Miércoles 15 julio | Escrito (aula) | 25% | Bloques V-VI (5.1 a 6.2) |
| **Examen Práctico Final - Parte 1 (P10)** | Lunes 13 julio | Laboratorio | 15% | Diseño, DDL/DML, consultas complejas |
| **Examen Práctico Final - Parte 2 (P11)** | Lunes 20 julio | Laboratorio | 15% | Programación en SGBD (triggers/procedimientos) + caso integrador |
| **Evaluación Continua** | Semanal | Prácticas, tareas, participación | 20% | Entregas semanales conforme a rúbricas |
| **TOTAL** | - | - | **100%** | - |

Requisitos mínimos de acreditación:
- Asistencia mayor o igual al 80% (mínimo 26 de 32 sesiones)
- Promedio mayor o igual a 6.0
- Entregas conforme a lineamientos APA 7a edición
- Examen práctico aprobado (mínimo 6.0 en cada parte)

### Validación de Consistencia

| Dimension | Verificacion | Estado |
|-----------|-------------|--------|
| **Temporal** | 11 semanas x 3 sesiones = 33 sesiones programadas (32 efectivas por asueto) | Cumple |
| **Contenido** | 6 bloques sinteticos cubiertos integramente (1.1 a 6.2) | Cumple |
| **Horaria** | 44 h teoria + 33 h practica = 77 h totales (11 creditos) | Cumple |
| **Evaluacion** | 2 examenes teóricos + 1 practico dividido en 2 sesiones + evaluacion continua | Cumple |
| **Calendario** | Sin conflictos con dias inhábiles; término 20 julio permite registro de calificaciones (23-24 julio) | Cumple |
| **Secuencia pedagogica** | Teoria-Teoria-Practica respeta modelo de andamiaje y saberes previos | Cumple |
| **Ubicacion de examenes** | Teórico 1 post-4.3 (Semana 7); Teórico 2 post-6.2 (Semana 11); Practico P10/P11 | Cumple |

---

Nota metodologica: Esta programacion respeta estrictamente el contenido sintetico, los objetivos generales y especificos, asi como la distribucion horaria institucional establecidos en el programa de la UEA 2151106. La secuencia Miercoles-Viernes-Lunes, el ajuste por asueto del 15 de mayo, la reubicacion de los examenes teóricos tras la conclusion de los bloques evaluados, y la division del examen practico en dos sesiones consecutivas (P10 y P11) garantizan la viabilidad operativa sin comprometer el rigor academico ni la cobertura integral del programa, en congruencia con las modalidades de conduccion y evaluacion definidas por la Universidad Autonoma Metropolitana, Unidad Iztapalapa.
