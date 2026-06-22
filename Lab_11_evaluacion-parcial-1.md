# EXAMEN TEÓRICO ACUMULATIVO — SEMANAS 1-6
**Unidad de Enseñanza-Aprendizaje:** Bases de Datos (2151106)  
**Licenciatura en Computación — UAM Iztapalapa**  
**Duración estimada:** 60 minutos  
**Total de reactivos:** 40 preguntas  

---

## UNIDAD 1: INTRODUCCIÓN A LAS BASES DE DATOS (10 preguntas)

**1. ¿Cuál es la definición más precisa de una base de datos en el contexto de los sistemas intensivos en software?**  
- A) Un conjunto de archivos planos almacenados en el sistema de archivos.  
- B) Una colección estructurada de datos persistentes, compartidos y controlados centralizadamente.  
- C) Un sistema operativo especializado en almacenamiento masivo.  
- D) Un lenguaje de programación para manipular información.  

**2. ¿Cuál es la principal diferencia entre un archivo tradicional y una base de datos?**  
- A) Los archivos son más rápidos que las bases de datos.  
- B) Las bases de datos eliminan la redundancia controlada y garantizan la integridad mediante restricciones.  
- C) Los archivos permiten concurrencia nativa sin bloqueo.  
- D) Las bases de datos no requieren sistema operativo para funcionar.  

**3. ¿Qué es un Sistema Administrador de Bases de Datos (SGBD o DBMS)?**  
- A) Un lenguaje de consultas estructurado.  
- B) Software que actúa como intermediario entre el usuario/aplicación y la base de datos, gestionando almacenamiento, seguridad y concurrencia.  
- C) Un tipo de hardware especializado para almacenamiento.  
- D) Un protocolo de red para transferencia de datos.  

**4. ¿Cuál de los siguientes NO es un modelo de datos clásico?**  
- A) Modelo jerárquico.  
- B) Modelo relacional.  
- C) Modelo de red.  
- D) Modelo de compilación.  

**5. En el modelo relacional, ¿cómo se representa la información?**  
- A) Mediante árboles y apuntadores.  
- B) Mediante tablas (relaciones) compuestas por tuplas y atributos.  
- C) Mediante grafos dirigidos.  
- D) Mediante documentos XML anidados.  

**6. ¿Qué significa la independencia lógica de los datos?**  
- A) Que los datos no pueden ser modificados.  
- B) Que la estructura conceptual de los datos puede cambiar sin afectar las aplicaciones que los usan.  
- C) Que los datos están cifrados y son inaccesibles.  
- D) Que cada usuario tiene su propia copia de los datos.  

**7. ¿Qué es la independencia física de los datos?**  
- A) Que los datos pueden almacenarse en diferentes dispositivos sin cambiar la estructura lógica.  
- B) Que los datos se eliminan automáticamente al apagar el sistema.  
- C) Que los datos solo pueden leerse en un tipo específico de hardware.  
- D) Que los datos no pueden ser respaldados.  

**8. ¿Cuál es el propósito principal del enfoque de base de datos frente al enfoque de archivos?**  
- A) Aumentar la redundancia para tener más copias de seguridad.  
- B) Integrar datos dispersos, reducir inconsistencias y compartir información de manera controlada.  
- C) Eliminar la necesidad de programadores.  
- D) Hacer que los datos sean temporales.  

**9. ¿Qué es un metadato en el contexto de bases de datos?**  
- A) Un dato de gran tamaño.  
- B) Datos sobre los datos: describen la estructura, restricciones y significado de la información almacenada.  
- C) Un dato cifrado.  
- D) Un dato que solo puede ser leído por el administrador.  

**10. ¿Cuál de las siguientes es una ventaja fundamental de utilizar un SGBD?**  
- A) Elimina por completo la necesidad de respaldos.  
- B) Permite control centralizado, minimiza la redundancia no controlada y garantiza la integridad.  
- C) Hace que los programas sean independientes del hardware y del sistema operativo.  
- D) Elimina la necesidad de administradores de bases de datos.  

---

## UNIDAD 2: SISTEMAS ADMINISTRADORES DE BASES DE DATOS (8 preguntas)  

**11. ¿Quién es el Administrador de Base de Datos (DBA)?**  
- A) El usuario final que consulta datos.  
- B) La persona responsable del diseño físico, implementación, mantenimiento, seguridad y respaldo de la base de datos.  
- C) El programador que escribe las aplicaciones.  
- D) El proveedor de hardware.  

**12. ¿Cuál de los siguientes es un componente principal de un SGBD?**  
- A) El navegador web.  
- B) El motor de almacenamiento y el procesador de consultas.  
- C) El sistema operativo de red.  
- D) El compilador de aplicaciones.  

**13. ¿Qué es el catálogo del sistema (o diccionario de datos)?**  
- A) Un listado de usuarios con sus contraseñas.  
- B) El repositorio donde el SGBD almacena los metadatos: definición de tablas, columnas, restricciones, índices y privilegios.  
- C) Un archivo de respaldo automático.  
- D) Una tabla de transacciones pendientes.  

**14. ¿Cuál es una desventaja potencial de utilizar un SGBD?**  
- A) Reduce la productividad de los programadores.  
- B) Introduce complejidad, costo de licenciamiento y requerimientos de hardware superiores.  
- C) Elimina la necesidad de personal capacitado.  
- D) Hace que los datos sean menos seguros.  

**15. ¿Qué tipo de usuario interactúa con la base de datos mediante consultas ad-hoc escritas en tiempo real?**  
- A) Usuario final ingenuo.  
- B) Usuario analítico o casual (sophisticated user).  
- C) Administrador de base de datos.  
- D) Programador de aplicaciones.  

**16. ¿Qué es el lenguaje DDL (Data Definition Language)?**  
- A) Lenguaje para consultar datos.  
- B) Lenguaje para definir la estructura de la base de datos: tablas, vistas, índices y restricciones.  
- C) Lenguaje para controlar transacciones.  
- D) Lenguaje para respaldar datos.  

**17. ¿Qué es el lenguaje DML (Data Manipulation Language)?**  
- A) Lenguaje para definir la estructura de la base de datos.  
- B) Lenguaje para insertar, actualizar, eliminar y consultar datos almacenados.  
- C) Lenguaje para administrar usuarios y permisos.  
- D) Lenguaje para crear respaldos.  

**18. ¿Cuál es la función del procesador de consultas (query processor) en un SGBD?**  
- A) Almacenar físicamente los datos en disco.  
- B) Traducir las consultas de alto nivel a operaciones de bajo nivel y optimizar su ejecución.  
- C) Generar respaldos automáticos.  
- D) Administrar los usuarios del sistema.  

---

## UNIDAD 3: DISEÑO DE BASES DE DATOS — MODELO ER Y RELACIONAL (14 preguntas)  

**19. ¿Qué es una entidad en el modelo Entidad-Relación (ER)?**  
- A) Un valor numérico.  
- B) Un objeto del mundo real, tangible o abstracto, que tiene existencia propia y puede distinguirse de otros objetos.  
- C) Una relación entre dos tablas.  
- D) Un atributo calculado.  

**20. ¿Qué es un atributo multivaluado?**  
- A) Un atributo que puede tener varios valores simultáneos para una misma entidad (ej. teléfonos).  
- B) Un atributo que es clave primaria.  
- C) Un atributo derivado de otro.  
- D) Un atributo obligatorio.  

**21. ¿Qué representa un rombo en un diagrama ER?**  
- A) Una entidad.  
- B) Un atributo.  
- C) Una relación entre entidades.  
- D) Una clave foránea.  

**22. ¿Qué indica la cardinalidad (1,1) en una relación?**  
- A) Que una instancia de una entidad se asocia con múltiples instancias de otra.  
- B) Que una instancia de una entidad se asocia con exactamente una instancia de la otra entidad, y viceversa.  
- C) Que no hay relación entre las entidades.  
- D) Que la relación es opcional.  

**23. ¿Qué es una clave primaria (primary key)?**  
- A) Un atributo que puede tener valores repetidos.  
- B) Un atributo o conjunto mínimo de atributos que identifica unívocamente a cada tupla en una relación.  
- C) Un atributo que referencia a otra tabla.  
- D) Un atributo calculado.  

**24. ¿Qué es una clave foránea (foreign key)?**  
- A) Una clave importada de otro sistema.  
- B) Un atributo en una relación que hace referencia a la clave primaria de otra relación, estableciendo un vínculo entre ambas.  
- C) Una clave que no se utiliza.  
- D) Una clave temporal.  

**25. ¿Cuál es la diferencia entre un esquema conceptual y un esquema físico?**  
- A) El conceptual describe la estructura lógica independiente del SGBD; el físico describe cómo se almacena en un SGBD específico.  
- B) El conceptual es más rápido que el físico.  
- C) El físico no requiere SGBD.  
- D) Son idénticos.  

**26. En la correspondencia entre el modelo ER y el modelo relacional, ¿cómo se traduce una relación 1:N?**  
- A) Se crea una tabla intermedia con las claves primarias de ambas entidades.  
- B) Se agrega la clave primaria de la entidad del lado "1" como clave foránea en la tabla del lado "N".  
- C) Se fusionan ambas entidades en una sola tabla.  
- D) Se elimina la relación.  

**27. ¿Cómo se traduce una relación M:N en el modelo relacional?**  
- A) Se agrega una clave foránea en una de las entidades.  
- B) Se crea una nueva tabla (tabla de asociación) que contiene las claves primarias de ambas entidades como clave foránea compuesta.  
- C) No se puede traducir.  
- D) Se convierte en dos relaciones 1:N.  

**28. ¿Qué es una dependencia funcional?**  
- A) La dependencia de una tabla respecto a otra.  
- B) La relación entre dos conjuntos de atributos A y B donde cada valor de A determina un único valor de B (A → B).  
- C) La dependencia de un atributo respecto al sistema operativo.  
- D) La relación entre dos bases de datos.  

**29. ¿Qué es la Primera Forma Normal (1FN)?**  
- A) Una relación donde todos los atributos son atómicos (sin grupos repetitivos ni multivaluados).  
- B) Una relación sin claves primarias.  
- C) Una relación con dependencias transitivas.  
- D) Una relación con claves foráneas.  

**30. ¿Qué es la Segunda Forma Normal (2FN)?**  
- A) Una relación en 1FN donde todos los atributos no clave dependen completamente de la clave primaria completa (no solo de parte de ella).  
- B) Una relación sin claves foráneas.  
- C) Una relación con atributos multivaluados.  
- D) Una relación sin atributos.  

**31. ¿Qué es la Tercera Forma Normal (3FN)?**  
- A) Una relación en 2FN donde no existen dependencias transitivas (los atributos no clave no dependen de otros atributos no clave).  
- B) Una relación sin claves primarias.  
- C) Una relación con claves compuestas.  
- D) Una relación con atributos derivados.  

**32. ¿Qué es la Forma Normal de Boyce-Codd (BCNF)?**  
- A) Una relación donde todo determinante es una superclave.  
- B) Una relación sin claves foráneas.  
- C) Una relación con atributos multivaluados.  
- D) Una relación sin atributos.  

---

## UNIDAD 4: DEFINICIÓN DE ESQUEMAS DE BASES DE DATOS (8 preguntas)

**33. ¿Qué sentencia SQL se utiliza para crear una tabla?**  
- A) INSERT TABLE  
- B) CREATE TABLE  
- C) DEFINE TABLE  
- D) NEW TABLE  

**34. ¿Qué restricción de integridad garantiza que una clave primaria no pueda tener valores NULL?**  
- A) CHECK  
- B) UNIQUE  
- C) NOT NULL y PRIMARY KEY  
- D) FOREIGN KEY  

**35. ¿Qué es la integridad referencial?**  
- A) La garantía de que todas las tablas tengan el mismo número de columnas.  
- B) La garantía de que toda clave foránea tenga un valor correspondiente en la tabla referenciada o sea NULL.  
- C) La garantía de que todos los datos estén cifrados.  
- D) La garantía de que las tablas no puedan eliminarse.  

**36. ¿Qué cláusula se utiliza para definir una clave foránea en SQL?**  
- A) PRIMARY KEY  
- B) UNIQUE  
- C) FOREIGN KEY ... REFERENCES  
- D) CHECK  

**37. ¿Qué es una vista (view) en SQL?**  
- A) Una tabla física que almacena datos permanentemente.  
- B) Una tabla virtual definida por una consulta, que no almacena datos físicamente (excepto vistas materializadas) y se utiliza para simplificar consultas o restringir acceso.  
- C) Un índice de búsqueda.  
- D) Un procedimiento almacenado.  

**38. ¿Cuál es la principal ventaja de utilizar vistas?**  
- A) Aumentan la velocidad de inserción de datos.  
- B) Proporcionan seguridad (ocultando columnas), simplificación de consultas complejas e independencia lógica.  
- C) Eliminan la necesidad de claves primarias.  
- D) Hacen que la base de datos ocupe menos espacio en disco.  

**39. ¿Qué restricción se utiliza para validar que un valor cumpla con una condición específica?**  
- A) UNIQUE  
- B) NOT NULL  
- C) CHECK  
- D) DEFAULT  

**40. ¿Qué sucede si se intenta eliminar una tabla que es referenciada por una clave foránea en otra tabla?**  
- A) Se elimina automáticamente la tabla referenciada.  
- B) El SGBD rechaza la operación por defecto para preservar la integridad referencial (a menos que se especifique CASCADE).  
- C) Se eliminan solo los datos, no la estructura.  
- D) Se convierte automáticamente en una vista.  

---

## HOJA DE RESPUESTAS (Para el Docente)

| # | Resp. | # | Resp. | # | Resp. | # | Resp. | # | Resp. |
|---|-------|---|-------|---|-------|---|-------|---|-------|
| 1 | B | 9 | B | 17 | B | 25 | A | 33 | B |
| 2 | B | 10 | B | 18 | B | 26 | B | 34 | C |
| 3 | B | 11 | B | 19 | B | 27 | B | 35 | B |
| 4 | D | 12 | B | 20 | A | 28 | B | 36 | C |
| 5 | B | 13 | B | 21 | C | 29 | A | 37 | B |
| 6 | B | 14 | B | 22 | B | 30 | A | 38 | B |
| 7 | A | 15 | B | 23 | B | 31 | A | 39 | C |
| 8 | B | 16 | B | 24 | B | 32 | A | 40 | B |

---

## DISTRIBUCIÓN TEMÁTICA Y COGNITIVA

| Unidad | Reactivos | % del Examen |
|--------|-----------|--------------|
| 1. Introducción a las BD | 1-10 | 25% |
| 2. SGBD | 11-18 | 20% |
| 3. Diseño (ER y Relacional) | 19-32 | 35% |
| 4. Definición de Esquemas | 33-40 | 20% |

**Niveles cognitivos evaluados (Taxonomía de Bloom):**
- **Recordar/Comprender:** ~60% (definiciones, conceptos, terminología)
- **Aplicar/Analizar:** ~35% (correspondencia ER-relacional, normalización, restricciones)
- **Evaluar:** ~5% (selección de mejores prácticas, integridad)

---

## RECOMENDACIONES DE APLICACIÓN

1. **Formato:** Puede aplicarse en plataforma LMS (Moodle, Canvas) o en papel.
2. **Tiempo:** 60 minutos es adecuado para 40 reactivos de opción múltiple.
3. **Criterio de aprobación sugerido:** 70% (28 aciertos).
4. **Retroalimentación:** Al ser un examen acumulativo previo al Laboratorio 11, los resultados pueden orientar la nivelación de estudiantes antes de enfrentar el proyecto de mitad de curso.

¿Desea que ajuste el número de reactivos, el nivel de dificultad, o que agregue preguntas sobre algún tema específico que haya cubierto con mayor profundidad en sus sesiones presenciales?
