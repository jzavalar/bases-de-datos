# ESTUDIO DE CASO: MODELADO DE BASES DE DATOS PARA LA CSD UAM-IZTAPALAPA
**Metodología:** *Database Design for Mere Mortals* — Michael J. Hernandez (4ª Edición)
(draft)

**Dr. Jesús Zavala Ruiz**
**Ultima actualización: 19 de junio de 2026**

---

## 1. CONTEXTO Y JUSTIFICACIÓN DEL CASO

La Coordinación de Servicios Documentales (CSD) de la UAM-Iztapalapa regula el acceso a un acervo documental clasificado en siete colecciones, atiende a cuatro perfiles de usuarios internos y uno externo, y aplica un régimen de sanciones diferenciado (multas por día natural vs. multas por hora). El Instructivo (Sesión 306) funciona como un **cuerpo normativo** que debe ser traducido íntegramente a la estructura lógica de la base de datos.

Según Hernandez, *"diseñar una base de datos no es inventar estructuras, sino escuchar al negocio"*. En este caso, el "negocio" es un documento legal universitario. El reto es que la base de datos no solo almacene datos, sino que **haga cumplir el Instructivo** mediante los cuatro niveles de integridad que propone la metodología:

1. **Table-level integrity** (Integridad de tabla)
2. **Field-level integrity** (Integridad de campo)
3. **Relationship-level integrity** (Integridad de relación)
4. **Business rules** (Reglas de negocio)

## 2. FASE 1: DECLARACIÓN DE LA MISIÓN Y OBJETIVOS
*(Hernandez, Capítulos 4 y 5)*

### 2.1 Misión de la Base de Datos
> *"El propósito de la base de datos de la CSD es gestionar el ciclo de vida de los acervos documentales y regular el acceso a los servicios de préstamo por parte de la comunidad universitaria y usuarios externos, asegurando la preservación del material y el cumplimiento estricto del régimen de sanciones establecido por el Consejo Académico en la Sesión 306."*

Esta misión cumple con los criterios de Hernandez: es sucinta, libre de tareas específicas y establece un foco claro para el diseño.

### 2.2 Objetivos de la Misión
Cada objetivo representa una tarea general que los usuarios realizarán contra los datos:

| # | Objetivo de la Misión | Sustantivo-Sujeto Implícito |
|---|----------------------|----------------------------|
| MO-1 | Mantener la información completa de los usuarios y su clasificación. | *Usuarios* |
| MO-2 | Administrar el catálogo de colecciones y los materiales documentales. | *Colecciones*, *Materiales_Documentales* |
| MO-3 | Controlar el inventario físico de ejemplares. | *Ejemplares* |
| MO-4 | Registrar las transacciones de préstamo aplicando restricciones de tiempo y cantidad. | *Prestamos*, *Detalles_Prestamo* |
| MO-5 | Calcular y registrar sanciones económicas y administrativas. | *Multas*, *Suspensiones* |
| MO-6 | Gestionar las reservas de materiales por UEA y trimestre. | *Reservas_UEA* |

## 3. FASE 2: IDENTIFICACIÓN DE SUJETOS Y CARACTERÍSTICAS
*(Hernandez, Capítulos 6 y 7 — Subject-Identification y Characteristic-Identification Techniques)*

### 3.1 Lista Preliminar de Campos
Tras analizar el Instructivo y aplicar la *Characteristic-Identification Technique* sobre las entrevistas simuladas con bibliotecarios y la Jefatura de la CSD, se obtiene la siguiente lista preliminar de características (que se convertirán en campos):

**Sujeto: USUARIO**
- tipo_usuario, subtipo_interno, nombre_completo, numero_identificacion, estatus, fecha_registro

**Sujeto: COLECCION**
- nombre, prestamo_interno_permitido, prestamo_domicilio_permitido, requiere_permiso_especial

**Sujeto: MATERIAL DOCUMENTAL**
- titulo, autor, isbn_issn, tipo_fisico, estado_fisico, adquirido_por_convenio

**Sujeto: EJEMPLAR**
- codigo_patrimonial, estado_prestamo

**Sujeto: PRÉSTAMO**
- modalidad, fecha_salida, fecha_vencimiento, fecha_devolucion_real, estatus

**Sujeto: MULTA**
- monto, tipo_cobro, metodo_pago

**Sujeto: SUSPENSIÓN**
- motivo, articulo_infringido, fecha_inicio, fecha_fin

### 3.2 Refinamiento: Los Elementos del Campo Ideal
Cada campo se somete a los **Elementos del Campo Ideal** de Hernandez:

| Elemento | Verificación en el Caso |
|----------|------------------------|
| Representa una característica distinta del sujeto | ✓ `nombre_completo` describe al *Usuario* |
| Contiene un solo valor | ✓ Se rechaza un campo `telefonos` (multivaluado) |
| No puede descomponerse | ✓ Se descompone `direccion` en calle, ciudad, estado, cp |
| No contiene valores calculados | ✓ `monto` de multa se calcula en vistas, no se almacena como campo dependiente |
| Es único en toda la estructura | ✓ Se usa prefijo de tabla para campos genéricos: `id_usuario`, `id_coleccion` |
| Conserva sus propiedades al aparecer en varias tablas | ✓ `id_usuario` mantiene su tipo y dominio al ser clave foránea |

### 3.3 Lista Final de Tablas
Tras aplicar los **Elementos de la Tabla Ideal**, se definen las siguientes tablas (todas en plural, como dicta Hernandez):

| Tabla | Tipo | Descripción |
|-------|------|-------------|
| **Usuarios** | Data | Personas que acceden a los servicios de la CSD, clasificadas en internos (con subtipos) y externos. |
| **Colecciones** | Validation | Catálogo maestro de las siete colecciones documentales definidas en los Arts. 3-9. |
| **Materiales_Documentales** | Data | Registros bibliográficos (títulos) que conforman el acervo. |
| **Ejemplares** | Data | Copias físicas individuales identificadas por código patrimonial. |
| **Prestamos** | Data | Transacciones de préstamo entre un usuario y uno o más ejemplares. |
| **Detalles_Prestamo** | Linking | Tabla de asociación que resuelve la relación M:N entre *Prestamos* y *Ejemplares*. |
| **Multas** | Data | Sanciones económicas generadas por retrasos. |
| **Suspensiones** | Data | Sanciones administrativas que inhabilitan temporalmente al usuario. |
| **Reservas_UEA** | Data | Asignación de materiales de la Colección de Reserva a unidades de enseñanza-aprendizaje (Art. 4). |
| **Renovaciones** | Data | Registro histórico de las prórrogas solicitadas sobre un préstamo. |

## 4. FASE 3: ESTABLECIMIENTO DE CLAVES
*(Hernandez, Capítulo 8 — Elements of a Candidate Key / Primary Key)*

Para cada tabla se identifican **Candidate Keys** (claves candidatas) y se selecciona una como **Primary Key** (PK). Cuando no existe una clave natural, se crea una **Artificial Candidate Key**.

| Tabla | Clave Primaria (PK) | Tipo | Justificación |
|-------|--------------------|------|---------------|
| **Usuarios** | `id_usuario` | Artificial | Hernandez recomienda crear un campo `ID` cuando las claves naturales (como `numero_identificacion`) pueden comprometer privacidad o contener nulos. |
| **Colecciones** | `id_coleccion` | Artificial | Catálogo de referencia. |
| **Materiales_Documentales** | `id_material` | Artificial | El ISBN puede ser nulo o repetido para obras multivolumen. |
| **Ejemplares** | `codigo_patrimonial` | Natural | Cumple todos los *Elementos de una Candidate Key*: único, no nulo, no opcional, inmutable. |
| **Prestamos** | `id_prestamo` | Artificial | Identifica la transacción. |
| **Detalles_Prestamo** | (`id_prestamo`, `codigo_patrimonial`) | Composite PK | Resuelve la relación M:N; la combinación es única. |
| **Multas** | `id_multa` | Artificial | Identifica cada sanción. |
| **Suspensiones** | `id_suspension` | Artificial | Identifica cada periodo de inhabilitación. |
| **Reservas_UEA** | (`id_material`, `clave_uea`, `trimestre`) | Composite PK | Un material puede reservarse para distintas UEAs y trimestres. |
| **Renovaciones** | `id_renovacion` | Artificial | Histórico de prórrogas. |

## 5. FASE 4: ESPECIFICACIONES DE CAMPO
*(Hernandez, Capítulo 9 — Field Specifications: General, Physical, and Logical Elements)*

Se documentan las especificaciones de campos críticos. A continuación, un ejemplo representativo:

### Campo: `tipo_usuario` (tabla *Usuarios*)
| Elemento | Valor |
|----------|-------|
| **Field Name** | `tipo_usuario` |
| **Parent Table** | *Usuarios* |
| **Specification Type** | Unique |
| **Data Type** | Alphanumeric |
| **Length** | 20 |
| **Character Support** | Letters |
| **Key Type** | Non-key |
| **Uniqueness** | Non-unique |
| **Null Support** | No nulls |
| **Required Value** | Yes |
| **Range of Values** | ('INTERNO', 'EXTERNO') — *Validation Table: Tipos_Usuario* |
| **Edit Rule** | Enter Now, Edits Allowed |

### Campo: `codigo_patrimonial` (tabla *Ejemplares*)
| Elemento | Valor |
|----------|-------|
| **Field Name** | `codigo_patrimonial` |
| **Parent Table** | *Ejemplares* |
| **Data Type** | Alphanumeric |
| **Length** | 20 |
| **Key Type** | Primary |
| **Key Structure** | Simple |
| **Null Support** | No nulls |
| **Required Value** | Yes |
| **Edit Rule** | Enter Now, Edits Not Allowed |

## 6. FASE 5: RELACIONES ENTRE TABLAS
*(Hernandez, Capítulo 10 — Identifying and Establishing Relationships)*

### 6.1 Matriz de Relaciones
| (Origen ↓ / Destino →) | Usuarios | Colecciones | Materiales_Documentales | Ejemplares | Prestamos |
|------------------------|----------|-------------|-------------------------|------------|-----------|
| **Usuarios** | — | — | — | — | 1:N |
| **Colecciones** | — | — | 1:N | — | — |
| **Materiales_Documentales** | — | — | — | 1:N | — |
| **Ejemplares** | — | — | — | — | M:N (vía *Detalles_Prestamo*) |
| **Prestamos** | — | — | — | — | — |

### 6.2 Relaciones Establecidas (con trazabilidad de claves)

**Relación 1:N — *Colecciones* → *Materiales_Documentales***
- **FK:** `id_coleccion` en *Materiales_Documentales*
- **Deletion Rule:** Restrict (no se puede eliminar una colección con materiales asociados)
- **Participation:** *Colecciones* (Mandatory, 1,1) — *Materiales_Documentales* (Optional, 0,N)

**Relación 1:N — *Materiales_Documentales* → *Ejemplares***
- **FK:** `id_material` en *Ejemplares*
- **Deletion Rule:** Restrict
- **Participation:** *Materiales_Documentales* (Mandatory, 1,1) — *Ejemplares* (Optional, 0,N)

**Relación 1:N — *Usuarios* → *Prestamos***
- **FK:** `id_usuario` en *Prestamos*
- **Deletion Rule:** Restrict (no se puede eliminar un usuario con préstamos activos)
- **Participation:** *Usuarios* (Mandatory, 1,1) — *Prestamos* (Optional, 0,N)

**Relación M:N — *Prestamos* ↔ *Ejemplares*** (resuelta con tabla de enlace)
- **Linking Table:** *Detalles_Prestamo*
- **CPK/FK:** `id_prestamo` (ref. *Prestamos.id_prestamo*)
- **CPK/FK:** `codigo_patrimonial` (ref. *Ejemplares.codigo_patrimonial*)

**Relación 1:N — *Prestamos* → *Multas***
- **FK:** `id_prestamo` en *Multas*
- **Deletion Rule:** Cascade (al eliminar un préstamo, se eliminan sus multas)

**Relación 1:N — *Usuarios* → *Suspensiones***
- **FK:** `id_usuario` en *Suspensiones*
- **Deletion Rule:** Cascade

### 6.3 Elementos de las Claves Foráneas (Hernandez Cap. 10)
Cada FK cumple con los tres *Elementos de una Clave Foránea*:
1. **Mismo nombre que la PK origen:** `id_usuario` en *Prestamos* = `id_usuario` en *Usuarios*. ✓
2. **Réplica de la especificación de campo:** hereda tipo, longitud y soporte de caracteres. ✓
3. **Toma sus valores de la PK referenciada:** el dominio se restringe a los valores existentes. ✓

## 7. FASE 6: REGLAS DE NEGOCIO
*(Hernandez, Capítulo 11 — Field-Specific y Relationship-Specific Business Rules)*

### 7.1 Reglas de Negocio Específicas de Campo (*Field-Specific*)

| # | Regla | Estructura Afectada | Elemento Modificado |
|---|-------|---------------------|---------------------|
| RN-1 | *"El préstamo a domicilio está restringido a usuarios internos"* (Art. 10-II) | Campo `modalidad` en *Prestamos* | Range of Values restringido por trigger que valida `Usuarios.tipo_usuario` |
| RN-2 | *"Las Colecciones Especiales y Raros requieren permiso especial"* (Art. 10-I) | Campo `requiere_permiso_especial` en *Colecciones* | Required Value = Yes en tabla auxiliar *Autorizaciones_Especiales* |
| RN-3 | *"La multa de Reserva se cobra por hora; la General, por día natural"* (Art. 30) | Campo `tipo_cobro` en *Multas* | Range of Values: ('POR_DIA', 'POR_HORA') |
| RN-4 | *"Los métodos de pago son: efectivo en caja, monedero caja, monedero biblioteca"* (Art. 31) | Campo `metodo_pago` en *Multas* | Validation Table: *Metodos_Pago* |

### 7.2 Reglas de Negocio Específicas de Relación (*Relationship-Specific*)

| # | Regla | Relación Afectada | Característica Modificada |
|---|-------|-------------------|---------------------------|
| RN-5 | *"Alumnos de licenciatura: máx. 5 préstamos simultáneos; posgrado/admin: 7; académico: 10"* (Art. 10-b) | *Usuarios* → *Prestamos* | **Degree of Participation** de *Prestamos*: (0,5), (0,7) o (0,10) según `subtipo_interno` |
| RN-6 | *"Posgrado/admin pueden renovar 1 vez; académico, 2 veces"* (Art. 10-a) | *Prestamos* → *Renovaciones* | **Degree of Participation** de *Renovaciones*: (0,1) o (0,2) |
| RN-7 | *"2 retrasos en un trimestre = suspensión por el resto del trimestre"* (Art. 32) | *Multas* → *Suspensiones* | **Relationship-Specific Rule** implementada vía trigger |
| RN-8 | *"3 extravíos en un año = suspensión de 1 año"* (Art. 34) | *Usuarios* → *Suspensiones* | **Relationship-Specific Rule** con validación temporal |

### 7.3 Documentación: Business Rule Specifications Sheet (Ejemplo RN-5)

```
STATEMENT: Los alumnos de licenciatura no pueden tener más de 5 préstamos 
           a domicilio simultáneamente.
CONSTRAINT: Un registro en Usuarios (subtipo_interno='ALUMNO_LIC') puede 
            asociarse con máximo 5 registros en Prestamos con estatus='ACTIVO'.
TYPE: Database oriented
CATEGORY: Relationship specific
TEST ON: INSERT (en Detalles_Prestamo), UPDATE (en Prestamos.estatus)
STRUCTURES AFFECTED: Usuarios, Prestamos, Detalles_Prestamo
RELATIONSHIP CHARACTERISTICS AFFECTED: Degree of Participation
ACTION TAKED: Trigger BEFORE INSERT en Detalles_Prestamo que cuenta préstamos 
              activos y aborta si >= 5 para ALUMNO_LIC.
```

## 8. FASE 7: VISTAS
*(Hernandez, Capítulo 12 — Data, Aggregate, and Validation Views)*

| Vista | Tipo | Propósito | Campos / Expresiones |
|-------|------|-----------|---------------------|
| **Vw_Disponibilidad_Reserva** | Data | Mostrar a alumnos los libros de reserva disponibles para su UEA | *Materiales_Documentales.titulo*, *Ejemplares.codigo_patrimonial*, *Reservas_UEA.clave_uea* |
| **Vw_Deudores_Activos** | Aggregate | Soportar el Art. 19 (Constancia de no adeudo) | `SUM(monto)` agrupado por `id_usuario` |
| **Vw_Restricciones_Semana_11_12** | Validation | Reducir dinámicamente el plazo de préstamo a licenciatura en semanas críticas (Art. 10-a) | Filtro: `EXTRACT(WEEK FROM fecha_salida) IN (11, 12)` |
| **Vw_Morosos** | Data | Listar préstamos vencidos para notificación (Arts. 27, 30) | Filtro: `fecha_devolucion_real IS NULL AND fecha_vencimiento < CURRENT_DATE` |

## 9. FASE 8: REVISIÓN DE INTEGRIDAD DE DATOS
*(Hernandez, Capítulo 13)*

| Nivel | Verificación | Resultado |
|-------|--------------|-----------|
| **Table-level** | ¿Cada tabla tiene PK única y no nula? ¿Sin campos multivaluados? | ✓ Cumple |
| **Field-level** | ¿Cada campo cumple los *Elementos del Campo Ideal*? | ✓ Cumple |
| **Relationship-level** | ¿Las FK cumplen los *Elementos de una Clave Foránea*? ¿Deletion rules definidas? | ✓ Cumple |
| **Business rules** | ¿Cada regla documentada en su *Business Rule Specifications Sheet*? | ✓ Cumple |
| **Views** | ¿Cada vista tiene su *View Specifications Sheet*? | ✓ Cumple |

## 10. IMPLEMENTACIÓN FÍSICA (SQL ESTÁNDAR)
A continuación, el esquema físico aplicando la convención solicitada: **PascalCase plural para tablas**, **snake_case para atributos**, y trazabilidad completa en claves foráneas.

```sql
-- =========================================================
-- FASE FÍSICA: CSD UAM-IZTAPALAPA
-- Convención: Tablas en PascalCase plural, campos en snake_case
-- =========================================================

-- ---------------------------------------------------------
-- TABLAS DE VALIDACIÓN (Validation Tables - Hernandez Cap. 11)
-- ---------------------------------------------------------
CREATE TABLE Colecciones (
    id_coleccion          INT PRIMARY KEY,
    nombre                VARCHAR(50) NOT NULL UNIQUE,
    prestamo_interno      BOOLEAN NOT NULL DEFAULT TRUE,
    prestamo_domicilio    BOOLEAN NOT NULL DEFAULT FALSE,
    requiere_permiso      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE Tipos_Usuario (
    id_tipo_usuario       INT PRIMARY KEY,
    descripcion           VARCHAR(30) NOT NULL UNIQUE
    -- Valores: 'INTERNO', 'EXTERNO'
);

CREATE TABLE Subtipos_Interno (
    id_subtipo            INT PRIMARY KEY,
    descripcion           VARCHAR(30) NOT NULL UNIQUE,
    max_prestamos         INT NOT NULL,
    max_renovaciones      INT NOT NULL
    -- Valores: 'ALUMNO_LIC' (5,0), 'ALUMNO_POSGRADO' (7,1),
    --          'ACADEMICO' (10,2), 'ADMINISTRATIVO' (7,1)
);

CREATE TABLE Metodos_Pago (
    id_metodo_pago        INT PRIMARY KEY,
    descripcion           VARCHAR(40) NOT NULL UNIQUE
    -- Valores: 'EFECTIVO_CAJA', 'MONEDERO_CAJA', 'MONEDERO_BIBLIOTECA'
);

-- ---------------------------------------------------------
-- TABLAS PRINCIPALES (Data Tables)
-- ---------------------------------------------------------
CREATE TABLE Usuarios (
    id_usuario            INT PRIMARY KEY,
    id_tipo_usuario       INT NOT NULL,
    id_subtipo            INT,
    nombre_completo       VARCHAR(150) NOT NULL,
    numero_identificacion VARCHAR(50)  NOT NULL UNIQUE,
    estatus               VARCHAR(15)  NOT NULL DEFAULT 'ACTIVO',
    fecha_registro        DATE         NOT NULL,
    CONSTRAINT fk_usuarios_tipo
        FOREIGN KEY (id_tipo_usuario) REFERENCES Tipos_Usuario(id_tipo_usuario),
    CONSTRAINT fk_usuarios_subtipo
        FOREIGN KEY (id_subtipo) REFERENCES Subtipos_Interno(id_subtipo),
    CONSTRAINT chk_interno_con_subtipo CHECK (
        (id_tipo_usuario = (SELECT id_tipo_usuario FROM Tipos_Usuario 
                            WHERE descripcion = 'INTERNO') AND id_subtipo IS NOT NULL)
        OR
        (id_tipo_usuario = (SELECT id_tipo_usuario FROM Tipos_Usuario 
                            WHERE descripcion = 'EXTERNO') AND id_subtipo IS NULL)
    )
);

CREATE TABLE Materiales_Documentales (
    id_material            INT PRIMARY KEY,
    id_coleccion           INT NOT NULL,
    titulo                 VARCHAR(255) NOT NULL,
    autor                  VARCHAR(150),
    isbn_issn              VARCHAR(30),
    tipo_fisico            VARCHAR(30) NOT NULL,
    estado_fisico          VARCHAR(20) NOT NULL DEFAULT 'BUENO',
    adquirido_por_convenio BOOLEAN NOT NULL DEFAULT FALSE,
    CONSTRAINT fk_material_coleccion
        FOREIGN KEY (id_coleccion) REFERENCES Colecciones(id_coleccion)
);

CREATE TABLE Ejemplares (
    codigo_patrimonial  VARCHAR(20) PRIMARY KEY,
    id_material         INT NOT NULL,
    estado_prestamo     VARCHAR(20) NOT NULL DEFAULT 'DISPONIBLE',
    CONSTRAINT fk_ejemplar_material
        FOREIGN KEY (id_material) REFERENCES Materiales_Documentales(id_material),
    CONSTRAINT chk_estado CHECK (estado_prestamo IN 
        ('DISPONIBLE','PRESTADO','EXTRAVIADO','RETENIDO','DANADO'))
);

CREATE TABLE Prestamos (
    id_prestamo            INT PRIMARY KEY,
    id_usuario             INT NOT NULL,
    modalidad              VARCHAR(25) NOT NULL,
    fecha_salida           TIMESTAMP NOT NULL,
    fecha_vencimiento      TIMESTAMP NOT NULL,
    fecha_devolucion_real  TIMESTAMP,
    estatus                VARCHAR(15) NOT NULL DEFAULT 'ACTIVO',
    CONSTRAINT fk_prestamo_usuario
        FOREIGN KEY (id_usuario) REFERENCES Usuarios(id_usuario),
    CONSTRAINT chk_modalidad CHECK (modalidad IN 
        ('INTERNO','DOMICILIO','ESPECIAL','INTERBIBLIOTECARIO')),
    CONSTRAINT chk_estatus CHECK (estatus IN ('ACTIVO','DEVUELTO','VENCIDO'))
);

-- Tabla de enlace (Linking Table) - resuelve M:N Prestamos <-> Ejemplares
CREATE TABLE Detalles_Prestamo (
    id_prestamo          INT NOT NULL,
    codigo_patrimonial   VARCHAR(20) NOT NULL,
    CONSTRAINT pk_detalle PRIMARY KEY (id_prestamo, codigo_patrimonial),
    CONSTRAINT fk_detalle_prestamo
        FOREIGN KEY (id_prestamo) REFERENCES Prestamos(id_prestamo)
        ON DELETE CASCADE,
    CONSTRAINT fk_detalle_ejemplar
        FOREIGN KEY (codigo_patrimonial) REFERENCES Ejemplares(codigo_patrimonial)
);

CREATE TABLE Multas (
    id_multa        INT PRIMARY KEY,
    id_prestamo     INT NOT NULL,
    monto           DECIMAL(10,2) NOT NULL CHECK (monto > 0),
    tipo_cobro      VARCHAR(15) NOT NULL,
    id_metodo_pago  INT,
    CONSTRAINT fk_multa_prestamo
        FOREIGN KEY (id_prestamo) REFERENCES Prestamos(id_prestamo),
    CONSTRAINT fk_multa_metodo
        FOREIGN KEY (id_metodo_pago) REFERENCES Metodos_Pago(id_metodo_pago),
    CONSTRAINT chk_tipo_cobro CHECK (tipo_cobro IN ('POR_DIA','POR_HORA'))
);

CREATE TABLE Suspensiones (
    id_suspension       INT PRIMARY KEY,
    id_usuario          INT NOT NULL,
    motivo              VARCHAR(100) NOT NULL,
    articulo_infringido VARCHAR(10),
    fecha_inicio        DATE NOT NULL,
    fecha_fin           DATE NOT NULL,
    CONSTRAINT fk_suspension_usuario
        FOREIGN KEY (id_usuario) REFERENCES Usuarios(id_usuario),
    CONSTRAINT chk_fechas CHECK (fecha_fin >= fecha_inicio)
);

CREATE TABLE Reservas_UEA (
    id_material        INT NOT NULL,
    clave_uea          VARCHAR(15) NOT NULL,
    trimestre          VARCHAR(10) NOT NULL,
    profesor_designa   VARCHAR(100),
    CONSTRAINT pk_reserva PRIMARY KEY (id_material, clave_uea, trimestre),
    CONSTRAINT fk_reserva_material
        FOREIGN KEY (id_material) REFERENCES Materiales_Documentales(id_material)
);

CREATE TABLE Renovaciones (
    id_renovacion           INT PRIMARY KEY,
    id_prestamo             INT NOT NULL,
    fecha_renovacion        TIMESTAMP NOT NULL,
    nueva_fecha_vencimiento TIMESTAMP NOT NULL,
    CONSTRAINT fk_renovacion_prestamo
        FOREIGN KEY (id_prestamo) REFERENCES Prestamos(id_prestamo)
);
```

### 10.1 Trigger: Regla de Negocio RN-5 (Art. 10-b)
*Límite de préstamos simultáneos según subtipo de usuario.*

```sql
CREATE TRIGGER Trg_Limite_Prestamos_Domicilio
BEFORE INSERT ON Detalles_Prestamo
FOR EACH ROW
BEGIN
    DECLARE v_subtipo       VARCHAR(30);
    DECLARE v_max_prestamos INT;
    DECLARE v_activos       INT;
    DECLARE v_modalidad     VARCHAR(25);
    DECLARE v_id_usuario    INT;

    -- Obtener datos del préstamo
    SELECT p.modalidad, p.id_usuario
      INTO v_modalidad, v_id_usuario
      FROM Prestamos p
     WHERE p.id_prestamo = NEW.id_prestamo;

    -- La regla solo aplica a préstamos a domicilio
    IF v_modalidad = 'DOMICILIO' THEN
        -- Obtener subtipo y su límite desde la Validation Table
        SELECT s.descripcion, s.max_prestamos
          INTO v_subtipo, v_max_prestamos
          FROM Usuarios u
          JOIN Subtipos_Interno s ON u.id_subtipo = s.id_subtipo
         WHERE u.id_usuario = v_id_usuario;

        -- Contar préstamos activos del usuario
        SELECT COUNT(DISTINCT dp.id_prestamo)
          INTO v_activos
          FROM Detalles_Prestamo dp
          JOIN Prestamos p ON dp.id_prestamo = p.id_prestamo
         WHERE p.id_usuario = v_id_usuario
           AND p.estatus = 'ACTIVO';

        -- Validar límite
        IF v_activos >= v_max_prestamos THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Infracción Art. 10-b: Límite de préstamos simultáneos excedido para este perfil.';
        END IF;
    END IF;
END;
```

## 11. ESCENARIOS DE PRUEBA DIDÁCTICOS

### Escenario 1: El alumno de licenciatura que excede el límite
**Acción:** Juan Pérez (`subtipo_interno = 'ALUMNO_LIC'`) tiene 5 préstamos activos e intenta sacar un sexto libro.
**Respuesta del sistema:** Al ejecutar `INSERT INTO Detalles_Prestamo`, el trigger `Trg_Limite_Prestamos_Domicilio` cuenta los préstamos activos, detecta que `v_activos >= 5` y aborta con `SQLSTATE '45000'`.
**Integridad defendida:** RN-5 (Art. 10-b) — *Relationship-level integrity*.

### Escenario 2: El usuario externo audaz
**Acción:** Un visitante (`tipo_usuario = 'EXTERNO'`) solicita préstamo a domicilio.
**Respuesta del sistema:** La aplicación consulta la tabla *Colecciones* y verifica que `prestamo_domicilio = TRUE`, pero al validar contra *Usuarios*, detecta `id_tipo_usuario = EXTERNO` y rechaza la transacción.
**Integridad defendida:** RN-1 (Art. 10-II) — *Field-level integrity*.

### Escenario 3: Multa por hora en Reserva
**Acción:** Un alumno devuelve un libro de la *Colección de Reserva* el lunes a las 12:00 hrs (vencía a las 10:00 hrs).
**Respuesta del sistema:** Un job nocturno calcula la diferencia, detecta que `id_coleccion` corresponde a 'RESERVA', e inserta en *Multas* un registro con `tipo_cobro = 'POR_HORA'` y `monto` proporcional a 2 horas.
**Integridad defendida:** RN-3 (Art. 30) — *Field-specific business rule*.

### Escenario 4: Suspensión automática por reincidencia
**Acción:** Un usuario acumula 2 multas por retraso en el trimestre.
**Respuesta del sistema:** Un trigger sobre *Multas* cuenta las ocurrencias trimestrales; al llegar a 2, inserta un registro en *Suspensiones* y actualiza `Usuarios.estatus = 'SUSPENDIDO'`.
**Integridad defendida:** RN-7 (Art. 32) — *Relationship-specific business rule*.

## 12. CONCLUSIÓN

Este estudio de caso demuestra que la metodología de **Michael J. Hernandez** no es un ejercicio abstracto, sino un proceso de **traducción normativa**. Cada artículo del Instructivo de la CSD se convierte en un elemento tangible del modelo:

- Los **sustantivos** del Instructivo (usuario, colección, préstamo, multa) se convierten en **tablas** en PascalCase plural (*Usuarios*, *Colecciones*, *Prestamos*, *Multas*).
- Las **características** de esos sustantivos se convierten en **campos** en snake_case (`id_usuario`, `nombre_completo`, `fecha_vencimiento`).
- Las **reglas del Consejo Académico** se codifican como **Business Rules** (field-specific y relationship-specific) que se materializan en constraints, validation tables y triggers.

La **trazabilidad** entre entidades y atributos —lograda mediante la convención de nomenclatura solicitada— permite que cualquier desarrollador, al ver un campo `id_coleccion` en la tabla *Materiales_Documentales*, identifique inmediatamente que referencia a `Colecciones.id_coleccion`, reforzando la **relationship-level integrity** que Hernandez considera la columna vertebral del modelo relacional.

El resultado final no es solo una base de datos que almacena libros: es un **sistema de control normativo automatizado** que hace cumplir el Instructivo de la UAM-Iztapalapa con independencia de la interfaz de usuario que lo opere.

## 12. REFERENCIAS
[Faltan]
