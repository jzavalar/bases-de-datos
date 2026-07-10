### Lectura 08. Model Driven Architecture en Sistemas Empresariales
### Estudio de Caso: Creación y Mantenimiento de Bases de Datos con iDempiere

---

### 1. Introducción

Este capítulo describe la Arquitectura Dirigida por Modelos (MDA, *Model Driven Architecture*) de iDempiere y la funcionalidad del Diccionario de Datos (*Data Dictionary*), componente fundamental del sistema.

En la mayoría de las aplicaciones empresariales tradicionales, los desarrolladores deben diseñar código y probar cada pantalla individualmente. Este enfoque puede resultar extremadamente consumidor de tiempo y generar inconsistencias en toda la aplicación en términos de interfaz de usuario, experiencia del usuario y funcionalidad. Además, puede dificultar significativamente que los usuarios aprendan nuevas áreas de una aplicación compleja como un sistema ERP.

iDempiere simplifica esta tarea mediante la implementación de un concepto avanzado: un diccionario de datos centralizado y activo. Las ventanas se generan dinámicamente en tiempo de ejecución a partir del diccionario de datos. Los desarrolladores simplemente definen las reglas sobre cómo se debe mostrar la ventana y bajo qué condiciones específicas. Esto resulta en ventanas que únicamente muestran los datos que el usuario necesita visualizar para una situación determinada.

Por ejemplo, si una orden de venta implica que el cliente retira la mercancía inmediatamente y realiza el pago en el acto, no es necesario mostrar campos como Regla de Envío (*Shipping Rule*), Regla de Facturación (*Invoice Rule*) o Término de Pago (*Payment Term*). Sin embargo, si una orden de venta implica el envío de mercancía al cliente con facturación diferida para pago en una fecha posterior, estos campos se vuelven necesarios. Además de proporcionar una interfaz de usuario consistente, el diccionario de datos permite que la visualización de datos y campos se adapte según el nivel de seguridad del usuario. En otras aplicaciones, esto requeriría la definición de múltiples ventanas separadas.

### 2. Contexto Histórico: De Compiere a iDempiere

Para comprender la profundidad arquitectónica de iDempiere, es imperativo remontarse a sus orígenes. La historia de este sistema es la de tres generaciones de software empresarial que han refinado, durante más de dos décadas, un mismo principio fundamental: la separación entre el modelo de datos y su representación en interfaz de usuario.

#### 2.1. Los Orígenes: Jorg Janke y el Nacimiento de Compiere (1999)

La génesis de lo que hoy conocemos como iDempiere se encuentra en la visión de **Jorg Janke**, un arquitecto de software alemán con más de 20 años de experiencia en sistemas ERP de clase mundial. A finales de la década de 1980, Janke trabajaba como arquitecto de software en la empresa alemana ADV/Orga, donde asesoró a SAP en la migración de su sistema ERP R/2 hacia una arquitectura cliente-servidor. Su recomendación fue reescribir completamente el R/2 para incorporar características multidimensionales (multi-empresa, multi-idioma, multi-moneda). Sin embargo, SAP optó por agregar capas adicionales a las estructuras existentes, decisión que generó ineficiencias y complejidades que persisten hasta hoy en muchas implementaciones de SAP.

Tras esta experiencia, Janke ocupó posiciones relevantes en otros proveedores de ERP, hasta que consiguió financiamiento de **Goodyear**, el fabricante multinacional de neumáticos, para desarrollar un ERP personalizado que aprovechara la funcionalidad multidimensional y permitiera personalizaciones fáciles basadas en principios de diseño que originalmente habían sido rechazados por SAP.

En **1999**, Janke fundó **Compiere Inc.** para desarrollar y brindar soporte a su nuevo producto, también llamado Compiere —palabra italiana que significa "comprender". El sistema fue diseñado desde sus cimientos como una aplicación empresarial impulsada por modelos (*model-driven*), donde un diccionario de datos centralizado y activo dictaba el comportamiento de la interfaz de usuario. Compiere se convirtió rápidamente en uno de los proyectos más exitosos de SourceForge, manteniéndose entre los 10 proyectos principales durante cuatro años consecutivos a partir de 2002, alcanzando un millón de descargas y 100 socios comerciales.

#### 2.2. La Prueba de los 5 Minutos: La Demostración Fundacional

Uno de los conceptos más poderosos que Janke promovió para demostrar el valor de la Arquitectura Dirigida por Modelos fue lo que se conoció como la **"prueba de los 5 minutos"** (*5-minute test*). En marzo de 2008, Compiere publicó una demostración en video titulada *"Easier Application Customization with Compiere"* que ilustraba este principio con una premisa contundente:

> *"In just five minutes users can grasp what Compiere means by 'customization without programming,' and 'easily change any thing at any time.'"*

La demostración mostraba cómo un usuario funcional —no un programador— podía, en apenas cinco minutos, modificar la estructura de una ventana empresarial: agregar campos, reorganizar su disposición, cambiar etiquetas, definir reglas de validación y alterar el comportamiento de la interfaz, todo ello sin escribir una sola línea de código. Esta capacidad era posible gracias al **Diccionario de Datos** (*Data Dictionary*), el corazón del sistema, que almacenaba no solo la estructura de los datos sino también las reglas de presentación, validación y seguridad.

La prueba de los 5 minutos no era meramente un ejercicio de marketing; era la validación empírica de un principio arquitectónico profundo: si el modelo de datos está suficientemente bien abstraído y centralizado, la personalización deja de ser una tarea de ingeniería de software para convertirse en una actividad de configuración empresarial. Este principio, que Janke había concebido a finales de los 90, anticipaba lo que la industria llamaría más tarde *low-code* o *no-code development platforms*.

#### 2.3. El Primer Cisma: De Compiere a ADempiere (2006)

El éxito de Compiere atrajo tanto a usuarios como a inversionistas. En **2006**, Compiere Inc. recibió financiamiento de capital de riesgo (*venture capital*) por 6 millones de dólares. Este evento marcó un punto de inflexión. La empresa comenzó a transicionar hacia un modelo de negocio más restrictivo, limitando progresivamente la funcionalidad de su versión open source para posicionar mejor su próxima solución cerrada, "Enterprise Compiere".

La comunidad de desarrolladores e integradores, que había crecido alrededor del proyecto durante años, percibió esta dirección como una traición a los principios del software libre. Según Ramiro Vergara, representante de asuntos públicos de ADempiere:

> *"The community was eager to have a direction and move forward, after suffering neglect from Compiere Incorporated for the last 5 years. The product we are delivering today is a case in point, for years the only alternative to upgrade between Compiere versions was to purchase a support agreement from Compiere Inc or one of its partners, the community offered to develop a tool but this and the great majority of enhancements developed by the community, were never accepted by Compiere Inc."*

El **1 de septiembre de 2006**, la comunidad ejecutó el *fork*: nació **ADempiere** —del verbo italiano *adempiere*, que significa "cumplir un deber" o "realizar una tarea". El proyecto fue liderado inicialmente por **Víctor Pérez** de e-Evolution, junto con un consejo de contribuyentes que incluía a integradores de múltiples países. En apenas tres semanas de trabajo comunitario, ADempiere lanzó su primera versión entregable el 22 de septiembre de 2006, alcanzando rápidamente el top 5 de los rankings de SourceForge.

ADempiere heredó íntegramente el Diccionario de Datos de Compiere y su arquitectura de aplicación dirigida por modelos. El objetivo era mantener el código 100% open source bajo licencia GPLv2, con gobernanza comunitaria y desarrollo colaborativo abierto.

#### 2.4. El Segundo Cisma: De ADempiere a iDempiere (2011) — La Revolución OSGi

Durante los años 2008-2010, dentro de la comunidad ADempiere surgieron propuestas para modernizar la arquitectura del sistema mediante un diseño modular basado en **OSGi** (*Open Services Gateway initiative*), un framework Java que permite la carga dinámica de componentes (*bundles*) en tiempo de ejecución. Algunos desarrolladores experimentaron con implementaciones usando Apache Felix y Equinox.

Sin embargo, las diferencias técnicas y de visión dentro de la comunidad ADempiere se profundizaron. En **2011**, dos figuras clave —**Carlos Ruiz** (colombiano, fundador de GlobalQSS) y **Heng Sin Low** (malasio, desarrollador principal)— decidieron crear un nuevo proyecto que implementara los cambios arquitectónicos que consideraban cruciales para el futuro del sistema.

**Carlos Ruiz** había comenzado su trayectoria con Compiere en 2005, cuando decidió dejar su empleo en una empresa de telecomunicaciones para enfocarse completamente en el ecosistema open source ERP. Fue cofundador de ADempiere en 2006 y su líder técnico hasta 2011. **Heng Sin Low**, por su parte, fue el responsable de portar ADempiere a PostgreSQL y el mantenedor principal del cliente web ZK.

La diferencia fundamental entre iDempiere 1.0 y ADempiere fue la **migración completa a la plataforma OSGi**, que permitió:

1. **Arquitectura de plugins**: El código específico pudo ser reempaquetado como plugins modulares, facilitando la personalización sin modificar el núcleo.  
2. **Mejor rendimiento**: Se reemplazó JBoss con Apache Tomcat (y posteriormente con Jetty en 2015), reduciendo significativamente la huella de memoria.  
3. **Sistema de build moderno**: Se adoptó Eclipse Buckminster (y más tarde Maven Tycho) para la gestión de dependencias y compilación.  
4. **Rediseño de la interfaz web**: Una actualización mayor del framework ZK (de 3.6 a 6.0, y posteriormente a 8.0 y 9.6) permitió un rediseño completo de la GUI web.  
5. **Gestión de IDs distribuidos**: Se implementó un generador UUID para manejar el cruce de identificadores entre diferentes instancias del sistema.  

La mayoría de la comunidad activa de desarrolladores migró a iDempiere, consolidándolo como la evolución natural de la línea Compiere/ADempiere. Desde entonces, iDempiere ha mantenido un ciclo de lanzamientos consistente y riguroso, llegando en 2023 a la versión 11 con soporte para OpenJDK 17, y continuando su evolución hasta la versión 13 Orion que utilizamos en este curso.

#### 2.5. Una Genealogía de Principios

Esta historia no es meramente anecdótica; ilustra un principio fundamental de la ingeniería de software: las buenas ideas arquitectónicas sobreviven a las organizaciones que las crean. El Diccionario de Datos concebido por Jorg Janke en 1999 —con su separación radical entre modelo y presentación— ha demostrado ser tan robusto que ha sobrevivido a tres generaciones de proyectos, dos forks comunitarios y más de dos décadas de evolución tecnológica.

Cuando en las siguientes secciones exploremos el Diccionario de Datos de iDempiere, estemos conscientes de que no estamos ante una invención reciente, sino ante la refinación de un principio arquitectónico probado en el tiempo: que la complejidad empresarial puede gestionarse mediante modelos de datos suficientemente expresivos y centralizados.

### 3. El Diccionario de Datos

El diccionario de datos de iDempiere constituye una parte integral de la aplicación. "Conoce" cómo acceder a los datos y cómo se relacionan entre sí. El diccionario de datos contiene definiciones completas de entidades de datos (tipo, validación, etc.), especificaciones sobre cómo se muestran (etiqueta en pantallas e informes, texto de ayuda, secuencia de visualización y posición relativa a otros campos), y las reglas de visualización aplicables. Las reglas de seguridad y control de acceso también se mantienen en este componente.

Los datos en tiempo de ejecución son sensibles al contexto. Por ejemplo, el sistema "sabe" que una venta de mostrador no tiene un término de pago asociado, por lo que no lo muestra. También reconoce que debe haber existencias disponibles incluso si el registro de inventario muestra cero (porque, por ejemplo, una recepción de materiales no ha sido procesada). Sin embargo, si el usuario cambia el tipo de transacción a una orden estándar, un término de pago se convierte en una parte obligatoria de la transacción y el sistema reconoce la situación de "fuera de stock".

Adicionalmente, las traducciones de la interfaz de usuario, así como de los documentos de Terceros (*Business Partners*), se gestionan en el diccionario de datos.

El Diccionario de Datos es extensible por el usuario y puede incluir reglas e información especificadas por el usuario. Esto permite a los usuarios autorizados agregar nuevas tablas, nuevas pantallas y campos adicionales a pantallas existentes. Todos los elementos agregados pueden listarse e informarse automáticamente utilizando la funcionalidad de reportes estándar disponible en toda la aplicación.

El Diccionario de Datos está compuesto por seis entidades principales:

1. **Elementos** (*Elements*)  
2. **Tabla y Columna** (*Table and Column*)  
3. **Ventana, Pestaña y Campo** (*Window, Tab and Field*)  
4. **Contexto** (*Context*)  
5. **Informes y Procesos** (*Reports and Processes*)  
6. **Formularios** (*Forms*)  

#### 3.1. Elementos

Los Elementos proporcionan una referencia centralizada para todos los campos. Cualquier campo definido para cualquier tabla en iDempiere tiene un Elemento correspondiente. Esto proporciona un etiquetado consistente de campos, visualización en informes y texto de ayuda unificado. También significa que un elemento necesita definirse solo una vez, aunque pueda utilizarse en cientos de tablas y múltiples ventanas (por ejemplo, el elemento "Organización").

Los datos del Elemento (Nombre, Texto de Impresión, Descripción y Comentario) se sincronizan automáticamente con los campos correspondientes tanto en tablas como en ventanas.

![Elemento](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Element.jpg?version=2&modificationDate=1227595721000)

Los Elementos se definen en la ventana "Elemento" (*Element*). Esta ventana es accesible cuando se ha iniciado sesión con un rol de Administrador del Sistema (*System Administrator*).

Cada Elemento debe tener un **Nombre de Columna de Base de Datos** (*DB Column Name*). Esta es la referencia que se utiliza al vincular un Elemento a un campo específico en una tabla. También se requieren un **Nombre** (*Name*) y un **Texto de Impresión** (*Print Text*). El Nombre es la etiqueta que se utilizará en cualquier ventana, formulario o formulario de parámetros donde se haga referencia al elemento. El Texto de Impresión es el encabezado de columna que se imprimirá en los informes. Puede haber instancias en las que se desee un Nombre de Impresión más corto para maximizar el diseño del informe. Por ejemplo, el Elemento `ISVendor` es un Booleano de 1 posición. Para un encabezado de informe, es posible que se desee imprimir 'Prov' en lugar de 'Proveedor'.

Los campos **Descripción** (*Description*) y **Comentario** (*Comment*) no son obligatorios, pero en la mayoría de los casos se recomienda proporcionar valores para estos campos. El campo Descripción es el texto que se mostrará en la 'ayuda emergente' (*tooltip*) cuando se coloque el cursor sobre un campo. El campo Comentario se mostrará como la ayuda en línea (F1) para ese campo.

La casilla de verificación **Activo** (*Active*) indica que este Elemento está activo y puede ser referenciado al definir columnas de tablas.

El **Tipo de Entidad** (*Entity Type*) tendrá por defecto "Definido por el Usuario" (*User Defined*) para cualquier registro que se agregue. Puede cambiarse a otro Tipo de Entidad que se haya definido. No se debe utilizar "iDempiere" o "Diccionario" (*Dictionary*) si se desea que las adiciones se preserven al migrar o actualizar el sistema.

La ventana Elemento también contiene una pestaña **Utilizado en Columna** (*Used in Column*). Esta muestra todas las columnas donde se hace referencia a este Elemento. Esto puede ser útil si se está considerando cambiar un elemento existente, ya que indicará a qué otras entidades afectarían los cambios.

#### 3.2. Tabla y Columna

##### 3.2.1. Tabla

Las **Tablas** (*Tables*) son la entidad sobre la cual se construyen las Ventanas. Puede definirse una tabla y sus columnas asociadas directamente en el Diccionario de Aplicaciones de iDempiere (*Application Dictionary*) mediante la interfaz de usuario y utilizarse esta definición para crear la tabla de base de datos correspondiente o la vista de base de datos (*database view*) en la base de datos subyacente. Alternativamente, puede crearse primero una tabla de base de datos o una vista de base de datos en la base de datos, e importarse su definición al Diccionario de Aplicaciones de iDempiere.

Las nuevas tablas pueden crearse abriendo la ventana **Tabla y Columna** (*Table and Column*), mientras se ha iniciado sesión con un rol de Administrador del Sistema.

![Tabla](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/table.jpg?version=1&modificationDate=1227595721000)

Se debe ingresar un **Nombre de Tabla de Base de Datos** (*DB Table Name*). Si se desea importar la definición de una tabla de base de datos o vista de base de datos ya creada en la base de datos, debe asegurarse de que el nombre proporcionado sea exactamente el mismo.

Si el objeto de base de datos subyacente es una vista, debe marcarse la casilla de verificación **Vista** (*View*).

También debe ingresarse un **Nivel de Acceso a Datos** (*Data Access Level*) (para Roles automáticos) y un **Tipo de Transacción** (*Transaction Type*). El Tipo de Transacción indicará si se requiere o no un valor explícito de Organización (distinto de *) cuando se guarda un registro en esta tabla.

Otros campos son opcionales. Algunos de estos tienen un impacto directo en la funcionalidad del diccionario de datos:

- Los campos **Ventana** (*Window*) y **Ventana de PO** (*PO Window*) indican qué ventana será el 'Objetivo de Zoom' (*Zoom Target*).
- La casilla de verificación **Alto Volumen** (*High Volume*), si está seleccionada, hará que la ventana que hace referencia a esta tabla muestre inicialmente una ventana de consulta. Si la casilla de verificación está deseleccionada, devolverá todos los registros activos.

Para importar una tabla de base de datos o vista de base de datos ya creada desde la base de datos, debe seleccionarse el botón **'Crear Columnas desde Base de Datos'** (*Create Columns from Database*) o el botón **'Importar esta vista desde BD'** (*Import this view from DB*). Para verificar si una definición de vista se importó correctamente, debe seleccionarse el botón **'Crear esta vista en BD'** (*Create this view in DB*) para recrear la vista en la base de datos utilizando la definición de vista importada.

##### 3.2.2. Columna

Las **Columnas** (*Columns*) se mostrarán como campos en las ventanas que hacen referencia a la tabla. Las columnas pueden ingresarse en el Diccionario de Datos y luego sincronizarse con la base de datos, o pueden crearse a partir de la tabla ya poblada en la base de datos. Generalmente, las nuevas tablas y sus respectivas columnas se pueblan en la base de datos mediante una herramienta (por ejemplo, SQL Loader), pero si se tiene una o dos columnas nuevas que agregar, puede ser más fácil crearlas en iDempiere y luego sincronizarlas con la base de datos.

![Columna de Tabla](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/TableColumn.jpg?version=1&modificationDate=1227595721000)

Los valores seleccionados y los datos ingresados en la columna determinarán cómo y cuándo se muestran los campos en las ventanas. Existe un refinamiento adicional que puede realizarse en la definición de Ventana, que se cubrirá en la siguiente sección.

El **Nombre de Columna de Base de Datos** (*DB Column Name*) es obligatorio (se sugiere utilizar el mismo nombre que el Elemento).

Debe seleccionarse el **Elemento** (*Element*) que corresponde a esta columna. Si se crean las columnas desde la base de datos y no existe un Elemento con el mismo nombre, el proceso creará automáticamente el Elemento.

Debe ingresarse un **Nombre** (*Name*). Puede dejarse en blanco la Descripción y el Comentario, ya que se sincronizarán con el Elemento cuando se guarde el registro.

Algunos de los otros campos en Columna que son utilizados por el Diccionario de Datos incluyen:

- **Campo de Referencia** (*Reference*): determina cómo se accede al campo. Por ejemplo, `TableDir` indica que el nombre de la tabla puede determinarse eliminando el `_ID` del nombre del campo. Un valor de `List` solicitará una Referencia para la Lista (se discutirá más adelante).  
- **Lógica Predeterminada** (*Default Logic*): permite definir reglas que poblarán el campo al crear un nuevo registro. Esta lógica puede incluir variables basadas en selecciones realizadas al momento del inicio de sesión o basadas en el usuario que inició sesión.  
- **Lógica de Obligatoriedad** (*Mandatory Logic*): permite definir las circunstancias en las que esta columna es requerida.  
- **Código de Llamada** (*Callout Code*): permite la ejecución de código cuando el usuario sale del campo con la tecla Tab. Esto es una consecuencia de entrada de datos y no debe utilizarse para validación (la cual debe ocurrir antes de que el usuario abandone el campo). Un ejemplo de una llamada (*callout*) está en la ventana de Orden de Venta. Cuando el usuario ingresa un Tercero (*Business Partner*), la llamada que se ejecuta actualiza otros campos en la ventana, como Lista de Precios, Regla de Entrega y Dirección del Tercero.  
- **Identificador** (*Identifier*): indica qué se muestra en el campo o lista desplegable cuando un usuario lo selecciona. Por ejemplo, cuando el usuario selecciona un Grupo de Terceros, se muestra el Nombre del Grupo de Terceros, no el ID del Grupo de Terceros. Es posible tener múltiples Identificadores definidos y los campos se mostrarán separados por un '_'.  
- **TableUID**: indica si la columna puede ser utilizada por alguien que no tiene acceso al identificador único interno de la tabla para identificar de manera única un registro en la tabla. Por ejemplo, cuando un usuario desea buscar una Ventana específica, no sabe cuál es el `AD_Window_ID`. En su lugar, utilizará el Nombre de la Ventana para identificar de manera única un registro de Ventana. De manera similar, cuando iDempiere está migrando un registro de Ventana, no puede utilizar el `AD_Window_ID` para identificar de manera única un registro de Ventana, ya que `AD_Window_ID` no garantiza hacer referencia al mismo registro en diferentes sistemas. En su lugar, utilizará el Nombre de la Ventana para ver si la Ventana ya existe en la instancia de destino. Si no hay un registro con el mismo Nombre de Ventana, entonces iDempiere insertará un nuevo registro con ese Nombre de Ventana. Si ya existe un registro con el mismo Nombre de Ventana, entonces iDempiere comparará los atributos de los dos registros y actualizará los de la instancia de destino si hay diferencias. Para marcar una columna como identificador único externo, debe establecerse el valor de TableUID en un entero mayor que cero. Tenga en cuenta que es posible que deba establecerse más de una columna como identificadores únicos externos para la tabla. Por ejemplo, una Pestaña tendrá tanto la Ventana Padre como el Nombre de Pestaña como identificadores únicos externos (los Nombres de Pestaña deben ser únicos para una Ventana dada, pero pueden duplicarse entre Ventanas). Por definición, no debe marcarse el ID de registro interno como identificador único externo (es decir, `AD_Window.AD_Window_ID` y `AD_Tab.AD_Tab_ID` NO son elegibles para ser identificadores únicos externos). Sin embargo, tenga en cuenta que `AD_Tab.AD_Window_ID` es elegible para ser un identificador único externo porque describe la Ventana Padre del registro AD_Tab.  
- **Columna de Selección** (*Selection Column*): indica qué campos se muestran en la ventana de búsqueda inicial si se selecciona Buscar. Todos los campos de una tabla están disponibles en la pestaña Búsqueda Avanzada.  

El botón **'Sincronizar Columna'** (*Synchronize Column*) debe seleccionarse si se están creando las columnas en la base de datos o si se ha cambiado parte de la definición de la tabla (por ejemplo, el Nombre de Restricción o se aumentó la Longitud del Campo).

##### 3.2.3. Componente de Vista y Columna de Vista

Los **Componentes de Vista** (*View Components*) contienen las cláusulas FROM, WHERE, GROUP BY y HAVING de la definición de la instrucción SELECT de una vista de base de datos. Si la vista de base de datos consta de múltiples instrucciones SELECT (unidas con una UNION), se necesita un Componente de Vista para cada instrucción SELECT.

![Componente de Vista](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ViewComponent.jpg?version=2&modificationDate=1255813450000)

Las **Columnas de Vista** (*View Columns*) contienen la cláusula SELECT de la definición de la instrucción SELECT de una vista de base de datos.

![Columna de Vista](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ViewColumn.jpg?version=2&modificationDate=1255813450000)

Como ejemplo, si la vista de base de datos se crea con la siguiente instrucción:

```sql
CREATE VIEW algunaVista ( algunVistaID, algunVistaNombre) AS 
SELECT algunID, algunNombre FROM algunaTabla WHERE algunAtributo = 'Y'
```

Entonces, la siguiente información se almacenará en las diversas pestañas de la ventana 'Tabla y Columna':

- La pestaña **'Tabla'** almacenará el nombre de vista `algunaVista` y el hecho de que esta tabla de iDempiere corresponde a una vista de base de datos.
- La pestaña **'Columna'** almacenará los nombres y tipos de datos de las columnas de vista `algunVistaID` y `algunVistaNombre`.
- La pestaña **'Componente de Vista'** almacenará la cláusula FROM `FROM algunaTabla` y la cláusula WHERE `WHERE algunAtributo = 'Y'` de la instrucción SELECT de la vista.
- La pestaña **'Columna de Vista'** almacenará las columnas `algunID`, `algunNombre` en la cláusula SELECT de la instrucción SELECT de la vista.

#### 3.3. Ventana, Pestaña y Campo

La ventana **Ventana, Pestaña y Campo** (*Window, Tab and Field*) define la presentación de tablas y columnas dentro de cada ventana. Cada Pestaña en una Ventana hace referencia a una sola Tabla. Los Campos en la Pestaña de una Ventana hacen referencia a las Columnas en la Tabla.

##### 3.3.1. Ventana

![Ventana](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Window.jpg?version=1&modificationDate=1227595721000)

La pestaña **Ventana** define cada ventana en el sistema. El **Nombre** (*Name*) y el **Tipo de Ventana** (*Window Type*) son obligatorios. El Nombre es lo que se muestra en el encabezado de la ventana. También es el Nombre que se muestra en el Menú. El sistema sincronizará el Nombre de la Ventana con la entidad Menú.

El **Tipo de Ventana** determina el comportamiento de la ventana cuando el usuario la abre. Por ejemplo:

- Si el Tipo de Ventana es **Mantener** (*Maintain*), entonces se recuperan todos los registros activos.  
- Si el Tipo de Ventana es **Transacción** (*Transaction*), entonces solo se mostrarán aquellos registros que fueron creados o actualizados hoy o que no están Completados.  

(Tenga en cuenta que todos los registros pueden recuperarse utilizando la funcionalidad de Búsqueda). Generalmente, al tratar con ventanas de transacción, por ejemplo, Facturas, no se desea recuperar facturas que tienen 2 años de antigüedad; se desea ver en qué se trabajó hoy y qué aún necesita trabajo adicional.

###### 3.3.1.1. Acceso a Ventana

La pestaña **Acceso a Ventana** (*Window Access*) define los Roles que tienen acceso a esta Ventana. Esto generalmente se define en la ventana Rol.

##### 3.3.2. Pestaña

![Pestaña](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Tab.jpg?version=1&modificationDate=1227595721000)

La pestaña **Pestaña** (*Tab*) define cada Pestaña dentro de una Ventana. Cada Pestaña hace referencia a una sola tabla. Todas las columnas de la tabla pueden mostrarse, pero generalmente se utiliza una selección específica de campos. Tenga en cuenta que la lógica de visualización y de solo lectura (junto con la lógica predeterminada definida para la Tabla/Columna) se evalúa al cargar la ventana.

En Pestaña, los campos obligatorios son **Nombre**, **Tabla**, **Secuencia** (*Sequence*) y **Nivel de Pestaña** (*Tab Level*). El Nombre indica qué se muestra en el texto de Ayuda y la Tabla determinará qué campos están disponibles para visualización, así como la tabla que se actualizará cuando se agregue, elimine o modifique un registro en esta pestaña de ventana.

La **Secuencia** determina el orden en que aparecerán las pestañas en la ventana. Por defecto, el sistema incrementará la Secuencia en un valor de 10 para cada nueva pestaña agregada.

El **Nivel de Pestaña** determina si la pestaña es un registro hijo de una pestaña anterior:

- Un Nivel de Pestaña de **0** indica que es la Pestaña Padre inicial.  
- Un Nivel de Pestaña de **1** indica que es hijo de la Pestaña Padre (por ejemplo, en la ventana Producto, la pestaña LMD es hija de la Pestaña Producto).  
- Un Nivel de Pestaña de **2** indica que es hijo de la Pestaña de Nivel 1 anterior (por ejemplo, Componente de LMD es hijo de la Pestaña LMD).  

Algunas de las otras características de la definición de Pestaña son:

- **Proceso** (*Process*): indica que un proceso que puede imprimir un documento estará habilitado para esta pestaña. Esto es lo que controlará si el Botón de Imprimir en la ventana está habilitado (por ejemplo, en la ventana Factura hay un proceso definido y el botón de imprimir está habilitado, y en la ventana Producto no hay un proceso definido y el botón de imprimir no está habilitado).  
- **Diseño de Fila Única** (*Single Row Layout*): si está seleccionada, indica que cuando se abra la ventana, se mostrará en modo de registro único (solo en clientes que lo soporten).  
- **Casillas de verificación de Pestaña Avanzada, Contable y de Traducción** (*Advanced, Accounting and Translation Tab*): indican si la visualización de esta pestaña estará controlada, junto con la seguridad, por la configuración definida en Preferencias.  
- **Lógica de Visualización y de Solo Lectura** (*Display and Read Only Logic*): permiten la definición de reglas de negocio para controlar la visualización y actualización de una pestaña. Por ejemplo, en la ventana Producto, la pestaña LMD tiene Lógica de Solo Lectura que previene la entrada de datos si la casilla de verificación LMD no está seleccionada.  

##### 3.3.3. Campo

![Campo](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Field.jpg?version=1&modificationDate=1227595721000)

La pestaña **Campo** (*Field*) define los Campos mostrados dentro de una pestaña. Los cambios realizados en la pestaña Campo se vuelven visibles después de reiniciar debido al almacenamiento en caché. Si la Secuencia es negativa, los registros se ordenan de forma descendente. Tenga en cuenta que el nombre, la descripción y la ayuda se sincronizan automáticamente si se mantienen centralizadamente.

El **Nombre** y la **Columna** son obligatorios. Las columnas disponibles para selección se basan en la tabla definida para la Pestaña. El Nombre, la Descripción y el Comentario se sincronizarán desde la definición de Columnas (que se ha sincronizado desde la definición de Elemento). Si, en una ventana específica, se desea utilizar una etiqueta de campo diferente o tener una definición de ayuda diferente, simplemente debe ingresarse el contenido deseado y deseleccionarse la casilla de verificación **Mantenido Centralmente** (*Centrally Maintained*). Esto prevendrá la sincronización de este campo específico desde los valores de Tabla/Columna.

Algunos otros atributos de los Campos que afectan la visualización son:

- **Mostrado** (*Displayed*): indica si el campo se mostrará. Si un campo no es necesario en una implementación específica, puede deseleccionarse esta bandera. Tenga en cuenta que si el campo es obligatorio, primero debe asegurarse de que haya un valor predeterminado definido. Además, si un campo no se muestra, eso puede comprometer el diseño del campo.  
- **Solo Lectura** (*Read Only*): indica que el campo se mostrará pero no podrá actualizarse. Por ejemplo, si no se desea que los usuarios actualicen la Lista de Precios en una Orden de Venta, debe establecerse el campo como Solo Lectura. El campo Lista de Precios es obligatorio, pero hay un valor predeterminado (ya sea del Tercero o del Nivel de Organización).  
- **Secuencia y Misma Línea** (*Sequence and Same Line*): determinan dónde se muestra en la Ventana.  
- **Enfoque Predeterminado** (*Default Focus*): indica que cuando se ingresa un nuevo registro, el cursor del usuario estará en este campo.  
- **Ocultar** (*Obscure*): permite ofuscar el campo (por ejemplo, mostrar solo los últimos 4 dígitos de un número de tarjeta de crédito). Así es como también se imprimirá en los informes. Sin embargo, se almacena como texto claro en la base de datos. Puede cifrarse campos para visualización y en la base de datos seleccionando **Cifrado** (*Encrypted*).  

#### 3.4. Contexto

El **Contexto** permite la definición de etiquetas de campo alternativas para ventanas dadas. Por ejemplo, el campo Tercero (*Business Partner*) significa cosas diferentes para el usuario en diferentes Contextos. En una Orden de Venta, el Tercero es 'Cliente'; en una Orden de Compra es 'Proveedor'. La ventana Contexto define los diferentes contextos para el sistema. Luego se definen diferentes etiquetas de campo para los Elementos apropiados y el Contexto se asigna a las Ventanas deseadas.

![Contexto](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Context.jpg?version=1&modificationDate=1227595721000)

Se definen varios Contextos (por ejemplo, Ventas, Compras y Solicitud). Pueden agregarse nuevos registros de Contexto si es necesario.

![Contexto de Elemento](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Element+Context.jpg?version=1&modificationDate=1227595721000)

A continuación, para el Elemento apropiado, se agregan nuevos Nombres, Texto de Impresión, Descripción y Comentario. No es necesario definir valores de contexto para todos los campos. El sistema utilizará las definiciones de elemento estándar si no se define ningún contexto.

Por último, debe seleccionarse el Contexto apropiado para la Ventana (por ejemplo, en Orden de Venta el Contexto se define como Ventas).

#### 3.5. Informes y Procesos

En iDempiere, los **Informes** (*Reports*) y los **Procesos** (*Processes*) son técnicamente la misma entidad. Ambos pueden tener un pre-proceso (por ejemplo, Selección de Parámetros) y ambos pueden tener salida (por ejemplo, Visor de Informes). Para un usuario, sin embargo, se ven como entidades separadas. Por esa razón, se diferencian en el Menú por el icono.

Para definir un Informe, debe abrirse la ventana **Informes y Procesos** (*Report and Process*).

##### 3.5.1. Informes

![Informe](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Report.jpg?version=1&modificationDate=1227595721000)

El **Nombre** es el Nombre que se mostrará en el Menú, así como en el encabezado del informe.

La **Descripción** y el **Comentario** se mostrarán en la ventana de confirmación que aparece cuando el usuario hace clic en el icono del informe en el menú. Esta es una buena manera de proporcionar información sobre para qué se utiliza el Informe o Proceso.

El **Nivel de Acceso a Datos** se utiliza para la generación automática de seguridad de Rol.

Debe seleccionarse la casilla de verificación **Vista de Informe** (*Report View*) para indicar que esto es un Informe. Esto asignará el Icono de Informe a este elemento de menú, así como mostrará los campos apropiados (por ejemplo, Vista de Informe) para definir adicionalmente el informe.

Deben ingresarse los campos restantes según corresponda. El único otro campo que es obligatorio es la **Vista de Informe**.

La casilla de verificación **Impresión Directa** (*Direct Print*) indica que el informe se imprimirá automáticamente cuando se ejecute el informe.

Si se desea, puede ingresarse un **Formato de Impresión** (*Print Format*) para utilizar cuando se genere este informe. Si no se selecciona ningún formato de impresión, se utilizará el formato predeterminado.

Debe seleccionarse la pestaña **Parámetro** (*Parameter*) para definir los parámetros para este Informe. Los campos disponibles para selección están restringidos a aquellos campos en la Vista de Informe seleccionada.

Los **Parámetros** permiten valores predeterminados, requeridos u opcionales, y rangos para valores o fechas.

Cualquier Parámetro que se utilice al generar el informe se imprime en el encabezado del Informe.

##### 3.5.2. Procesos

Como se mencionó anteriormente, los Procesos se definen de manera similar a los Informes.

![Proceso](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Process.jpg?version=1&modificationDate=1227595721000)

Como la casilla de verificación Vista de Informe no está seleccionada, se utilizará el icono de Proceso en el Menú. Además, los campos Vista de Informe, Formato de Impresión e Impresión Directa no se muestran (este es un ejemplo de lógica de visualización definida para la ventana).

Aquí se ingresará un **Nombre de Clase** (*Classname*) y/o **Procedimiento** (*Procedure*) que está asociado con el Proceso. También puede seleccionarse **Proceso de Servidor** (*Server Process*) si esto debe ejecutarse solo en el Servidor (en oposición al cliente cuando se utiliza un cliente thick).

El campo **Flujo de Trabajo** (*Workflow*) se utiliza cuando el Proceso que se está definiendo es un proceso que se inicia mediante la selección de un botón (por ejemplo, el botón Completar en un documento).

Al igual que los Informes, los Procesos también pueden tener Parámetros para que el usuario los ingrese al ejecutar el proceso.

#### 3.6. Formularios

Los **Formularios** (*Forms*) son ventanas complejas que están codificadas de forma fija (*hard-coded*). Generalmente involucran datos de más de una tabla y pueden tener una relación uno a muchos o muchos a muchos. A diferencia de otras entidades, los formularios se definen por separado en diferentes clientes (ZK, Swing, etc.).

![Formulario](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/Form.jpg?version=1&modificationDate=1227595721000)

- El **Nombre de Clase** (*Classname*) es el código que se utiliza para generar el formulario para el cliente correspondiente.  
- El **Nombre de Clase Java para la Interfaz Web** (*Java Classname for the WebUI*) es el código que se utiliza para generar el formulario para la interfaz de usuario web (ZK).  
- Si está definido, la **URL JSP** define la URL que se utiliza para ejecutar la clase Java para la interfaz de usuario web.  

#### 3.7. Lógica de Negocio: Referencias y Validaciones

##### 3.7.1. Referencia

La **Referencia** (*Reference*) se utiliza en Columna (de Tabla y Columna) para controlar qué se muestra en un campo y cómo se muestra. Las Referencias pueden ser de uno de tres **Tipos de Validación** (*Validation Type*) diferentes: Tipo de Dato (*DataType*), Lista (*List*) o Validación de Tabla (*Table Validation*). El tipo seleccionado generalmente se basa en cómo se utiliza el campo y el nivel de control deseado.

Un **Tipo de Validación de Tipo de Dato** se utiliza para definir diferentes tipos de campo (por ejemplo, Botón, Fecha y Hora, Número). Generalmente, no se necesitará crear nuevas Referencias de un Tipo de Validación de Tipo de Dato.

A continuación se presenta una lista de los tipos de datos estándar admitidos en iDempiere:

- **Cuenta** (*Account*): Se utiliza para campos que almacenan combinaciones de cuentas. Un ejemplo de esto es el campo Combinación de Cuentas en la línea de Diario de Libro Mayor.  
- **Monto** (*Amount*): Es un campo numérico con 4 decimales utilizado para representar Montos. Un ejemplo de esto es el campo Total General en Órdenes de Venta.  
- **Asignación** (*Assignment*): Se utiliza para asignaciones de recursos. Un ejemplo de esto es el campo Asignaciones de Recursos en la línea de Orden de Venta.  
- **Binario** (*Binary*): Se utiliza para almacenar Datos Binarios (Blobs). Por ejemplo, se utiliza para almacenar adjuntos en la tabla AD_Attachment.  
- **Botón** (*Button*): Se utiliza para mostrar el campo como un botón en el que se puede hacer clic. Esto puede utilizarse para lanzar un proceso. Un ejemplo de esto es el botón "Copiar Líneas" en la ventana de Orden de Venta.  
- **Costos+Precios** (*Costs+Prices*): Se utiliza para mostrar campos numéricos. Se mostrará con la precisión mínima de moneda, o más.  
- **Fecha** (*Date*): Se utiliza para mostrar campos de fecha (sin marca de tiempo).  
- **Fecha+Hora** (*Date+Time*): Se utiliza para mostrar campos de fecha con marcas de tiempo.  
- **Nombre de Archivo** (*FileName*): Se utiliza para mostrar un nombre de archivo local.  
- **ID**: Es la clave primaria para cada tabla. Por ejemplo, la columna `C_Order_ID` tiene un tipo de referencia de ID. Para cada registro, iDempiere generará automáticamente un valor único para esta columna utilizando una secuencia interna.  
- **Imagen** (*Image*): Se utiliza para permitir a los usuarios agregar imágenes a las ventanas.  
- **Entero** (*Integer*): Se utiliza para representar valores enteros.  
- **Lista** (*List*): Se utiliza para permitir a los usuarios seleccionar desde un cuadro de lista desplegable que contiene una lista de valores.  
- **Ubicación (Dirección)** (*Location (Address)*): Se utiliza para campos que contienen información de dirección. Para estos campos, se permitirá al usuario abrir una ventana emergente donde puede ingresar información de dirección de calle. También hay un enlace desde esta ventana emergente a servicios de mapas.  
- **Localizador (Almacén)** (*Locator (WH)*): Se utiliza para mostrar una lista desplegable de localizadores apropiados del almacén.  
- **Memo**: Se utiliza para campos de texto grandes de hasta 2000 caracteres.  
- **Número** (*Number*): Se utiliza para cualquier campo numérico.  
- **Nombre de Impresora** (*Printer Name*): Se utiliza para permitir a los usuarios elegir entre una lista de impresoras accesibles.  
- **Atributo de Producto** (*Product Attribute*): Se utiliza para abrir una ventana emergente generada dinámicamente basada en los atributos que se han agregado para un producto dado. Un ejemplo de esto es el campo Atributos de Producto en la línea de Orden de Venta.  
- **Cantidad** (*Quantity*): Se utiliza para campos de cantidad. Un ejemplo de esto es el campo Cantidad en la línea de Orden de Venta.  
- **Búsqueda** (*Search*): Se utiliza para abrir una ventana emergente (ventana de búsqueda) donde el usuario puede ingresar diversos criterios para buscar el valor apropiado. Un ejemplo de esto es el campo Producto en la línea de Orden de Venta.  
- **Cadena** (*String*): Se utiliza para campos de cadena alfanumérica.  
- **Tabla** (*Table*): Se utiliza para referencias de clave foránea a otras entidades en el sistema. Se renderizará como un cuadro de diálogo desplegable con las columnas identificadoras de la tabla referenciada. Si se utiliza este tipo de dato, también debe especificarse la clave de referencia. La clave de referencia detalla la tabla que se está referenciando. Un ejemplo de esto es la columna `Bill_BPartner_ID` en la tabla C_Order (utilizada para mostrar el Facturar a en la ventana de Orden de Venta).  
- **Tabla Directa** (*Table Direct*): Es similar a las referencias de Tabla en que se utiliza para referencias de clave foránea a otras entidades en el sistema. La principal diferencia es que no tiene que especificarse la tabla referenciada utilizando la clave de referencia. En su lugar, la tabla se deriva automáticamente eliminando el "_ID" del nombre de la tabla. Por ejemplo, esto se utiliza para la columna `M_Warehouse_ID` en la tabla C_Order (utilizada para mostrar el Almacén en la ventana de Orden de Venta). En este caso, la tabla referenciada se deriva del nombre de la columna como M_Warehouse (al eliminar el "_ID" del nombre "M_Warehouse_ID" para obtener M_Warehouse). Esto da como resultado que el usuario vea una lista desplegable de todos los almacenes válidos.  
- **Texto** (*Text*): Se utiliza para Cadenas de Caracteres de hasta 2000 caracteres.  
- **Texto Largo** (*Text Long*): Se utiliza para Cadenas de Caracteres de más de 2000 caracteres.  
- **Hora** (*Time*): Se utiliza para representar Marcas de Tiempo. Un ejemplo de esto es el campo "Inicio de Ranura" en la ventana "Tipo de Recurso".  
- **URL**: Se utiliza para representar URLs. El usuario podrá enlazar a la URL utilizando el icono de lupa en el campo.  
- **Sí-No** (*Yes-No*): Se utiliza para representar campos con una lista desplegable que contiene los valores "Sí", "No" y una cadena vacía. Si esta columna se hace obligatoria, este campo se renderizará como una casilla de verificación en la interfaz web.  

Un **Tipo de Validación de Referencia de Validación de Tabla** se utiliza cuando se desea presentar al usuario una lista de valores para su selección y esa lista se basa en una tabla a la que el usuario puede o no tener la capacidad de agregar registros.

![Referencia de Tabla](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ReferenceTable.jpg?version=1&modificationDate=1227595721000)

Cuando se selecciona un Tipo de Validación de Validación de Tabla, la pestaña **Validación de Tabla** (*Table Validation*) se habilita (este es un ejemplo de lógica de solo lectura definida para una Pestaña en Ventana, Pestaña y Campo).

![Validación de Tabla de Referencia](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ReferenceTableVal.jpg?version=1&modificationDate=1227595721000)

Aquí se selecciona el **Nombre de Tabla** y la **columna Clave** para la Tabla. La **columna de Visualización** es lo que se mostrará en el cuadro de lista desplegable.

Debe seleccionarse la casilla de verificación **Mostrar Valor** (*Display Value*) para mostrar el valor del campo en el cuadro de lista desplegable.

Debe seleccionarse la casilla de verificación **Mostrar Identificadores** (*Display Identifiers*) si la lista debe mostrar todos los campos que están indicados como identificadores para la tabla.

Si es apropiado, debe ingresarse una cláusula SQL **WHERE** y **ORDER BY** para controlar qué se muestra en la lista y en qué orden. Esto se utiliza para evitar que las organizaciones resumen aparezcan en el cuadro de lista desplegable de organización en documentos.

Un **Tipo de Referencia de Lista** se utiliza cuando se desea presentar al usuario una lista de valores de la cual hacer una selección, pero esa lista se define en Referencia (en oposición a derivarse de una tabla). Las Listas se utilizan con mayor frecuencia en situaciones donde hay alguna lógica asociada con el valor seleccionado y, por lo tanto, hay código involucrado. En esas situaciones, se necesita saber cuáles podrían ser los valores posibles. Ejemplos de esto son la Regla de Envío en Orden de Venta. Estos valores están restringidos a la lista definida en Referencia y el valor seleccionado impacta el proceso de Generar Envío. Por lo tanto, no sería apropiado permitir que los usuarios ingresen cualquier valor que deseen.

![Lista de Referencia](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ReferenceList.jpg?version=1&modificationDate=1227595721000)

Cuando se selecciona un Tipo de Validación de Validación de Lista, la pestaña **Validación de Lista** (*List Validation*) se habilita. Esto permite la entrada de los valores que se mostrarán al usuario para su selección.

La **Clave de Búsqueda** (*Search Key*) se utiliza para controlar el orden en que aparecen los valores en la lista, mientras que el **Nombre** es lo que se presenta al usuario para su selección.

##### 3.7.2. Regla de Validación

Las **Reglas de Validación** (*Validation Rules*) se utilizan para controlar qué se muestra para selección a un usuario (similar a una cláusula WHERE de SQL en una entrada de Referencia). Sin embargo, estas pueden utilizarse para cualquier tipo de campo en una Columna. Se selecciona en la **Regla de Validación Dinámica** (*Dynamic Validation Rule*) en Tabla y Columna. Como lo indica el nombre, se ejecuta dinámicamente al seleccionar la lista o búsqueda.

![Regla de Validación](https://web.archive.org/web/20200217im_/http:/wiki.compiere.com/download/attachments/4260403/ValidationRule.jpg?version=1&modificationDate=1227595721000)

En este ejemplo, la validación se está utilizando para asegurar que la selección presentada al usuario contenga solo Ubicaciones que estén definidas para el Tercero seleccionado que tengan marcada la casilla de verificación Dirección de Envío. Esto previene la visualización de ubicaciones que están asociadas con otros Terceros o ubicaciones que no están indicadas como Direcciones de Envío.

Esta validación es dinámica porque se basa en el valor seleccionado en otro campo (en este caso, Tercero).

Al igual que con otras ventanas del Diccionario de Datos en iDempiere, la Regla de Validación tiene una pestaña **Utilizado en Columna** (*Used in Column*). Esto es útil si se desea cambiar una Regla de Validación, ya que puede verse desde esta pestaña en qué otras áreas del sistema se utiliza.

### 4. Resumen del Diccionario de Datos

El Diccionario de Datos en iDempiere es una poderosa herramienta de desarrollo. Gran parte de lo que se hace en código en un sistema tradicional puede lograrse en el Diccionario de Datos. Esto ayuda a proporcionar consistencia en toda la aplicación (especialmente cuando intervienen múltiples desarrolladores) y estabilidad, además de un desarrollo rápido y ágil.

La aplicación iDempiere está llena de ejemplos que muestran cómo funcionan las diferentes características del diccionario. No importa qué tipo de extensión o personalización se esté trabajando, generalmente puede encontrarse una instancia en otra parte de la aplicación que funcione de manera similar. Se sugiere utilizar estos como referencia en los propios esfuerzos de desarrollo.

### 5. Editor Visual del Diccionario de Datos

#### 5.1. Introducción y Visión General

El Editor Visual del Diccionario de Datos (*Visual Dictionary Editor*) de iDempiere proporciona una interfaz moderna para personalizar las ventanas de configuración de datos o de transacciones. Utiliza la función de **arrastrar y soltar** (*drag-n-drop*) para diseñar el diseño de la ventana exactamente como se desee. 

Con el Editor Visual puede:
- Personalizar las etiquetas de los campos  
- Ocultar campos que no se deseen  
- Pre-poblar valores de campos  
- Marcar campos como obligatorios o de solo lectura  
- Actualizar atributos de columnas como obligatorio, solo lectura, actualizable, valor predeterminado y más  
- Agregar nuevos campos a las ventanas junto con las columnas subyacentes de la base de datos en un flujo de usuario sin interrupciones  

El Editor Visual del Diccionario de Datos muestra la vista del usuario de la ventana mientras se trabaja en ella, haciendo que el diseño sea intuitivo y libre de errores.

El Editor Visual del Diccionario de Datos está disponible en la interfaz web de iDempiere. Se necesita tener el rol de Administrador del Sistema o privilegios equivalentes para acceder a esta función. Solo las ventanas que tienen un tipo de entidad distinto a "Diccionario" están disponibles para diseñar en el Editor Visual.

Para personalizar una ventana que entrega iDempiere:

1. Cree una nueva ventana en la ventana **Ventana, Pestaña y Campo** con tipo de entidad "Mantenida por el Usuario" (*User Maintained*)  
2. Haga clic en el botón **Copiar Pestañas de Ventana** (*Copy Window Tabs*) para copiar todas las pestañas y campos de la ventana original  
3. Complete el **Área de Contexto** en la definición de la ventana si es necesario  
4. Ejecute el proceso **Sincronizar Terminología** (*Synchronize Terminology*)  
5. Agregue la nueva ventana al menú apropiado  
6. Ejecute el proceso **Actualizar Acceso de Roles** (*Role Access Update*) para otorgar privilegios a la ventana  
7. Inicie el Editor Visual y seleccione la nueva ventana para cambiar su diseño y las propiedades de columna y campo  
8. Guarde sus cambios y ejecute el proceso **Restablecer Caché Web** (*Cache Reset Web*)  
9. Seleccione su ventana recientemente personalizada desde el menú para ver su nuevo diseño y lógica de negocio  

**Importante:** La funcionalidad para actualizar propiedades de columnas o agregar nuevos campos está disponible en versiones recientes de iDempiere. Las propiedades de diseño y campo pueden actualizarse en la mayoría de las versiones actuales.

![Editor Visual del Diccionario de Datos](https://web.archive.org/web/20190210185802im_/http:/wiki.compiere.com/download/attachments/4685832/CADE2.JPG?version=5&modificationDate=1268055025000)

El Editor Visual contiene **4 secciones principales**:

1. **La Barra de Herramientas** (*Toolbar*) (parte superior): donde se selecciona la ventana que se desea editar, junto con los iconos de guardar y deshacer para los cambios que se realicen.

2. **El panel Navegador** (*Navigator panel*) (izquierda): muestra un árbol con la ventana y las pestañas que incluye, mostradas en orden jerárquico, y todos los campos para la pestaña seleccionada. Hay un icono junto a cada campo que representa el tipo de visualización heredado de la referencia de columna vinculada (por ejemplo, Búsqueda, Casilla de verificación, Botón).

3. **Los paneles Propiedades de Campo y Propiedades de Columna** (*Field Properties and Column Properties panels*): muestran las propiedades del campo que se selecciona y su columna vinculada. Aquí es donde se define la lógica de negocio para el campo y la columna.

4. **El área Registro Único** (*Single Record area*): permite diseñar el diseño de formulario de 2 columnas para la pestaña seleccionada actualmente. La **Vista de Cuadrícula** (*Grid View*) permite reorganizar el orden en que deben mostrarse las columnas en el modo de vista de cuadrícula.

#### 5.2. Diseño de Ventana

Para utilizar el Editor Visual:

1. Inicie el Editor Visual e ingrese el nombre de la ventana que desea editar  
2. Solo puede diseñarse el layout para una pestaña a la vez  
3. El panel Navegador muestra todos los campos para la pestaña seleccionada actualmente  

Algunos campos están en **negro**, lo que indica que están disponibles para arrastrar desde el Navegador al área Registro Único para que puedan agregarse al diseño de la ventana. Otros campos están en **gris**, lo que indica que ya están incluidos en el diseño de la ventana y no están disponibles para arrastrar.

Puede:
- Arrastrar y soltar campos  
- Reposicionarlos entre filas o a un espacio vacío en la columna derecha buscando marcadores de inserción azules  
- Soltar campos sobre otros campos indicados por un resaltado amarillo y dejar que el diseño se reorganice automáticamente  

El diseño que se define aquí es exactamente la forma en que se verá la ventana en la aplicación iDempiere:
- Los campos que son **obligatorios** se muestran en **amarillo**  
- Los campos que son de **solo lectura** se muestran en **gris**  

Si se desea ocultar un campo para que no se muestre en la ventana, debe arrastrarse fuera del diseño Registro Único hacia el Navegador.

Para definir el orden de las columnas:
- Cámbiese al área **Vista de Cuadrícula**  
- Arrástrense y suéltense campos para definir el orden de las columnas  
- Solo pueden reposicionarse campos aquí  
- Si se desea agregar u ocultar campos, debe hacerse en el área **Registro Único**  

Si el usuario con el que se inició sesión tiene **AutoCommit** activado en Preferencias de Usuario, los cambios se guardarán automáticamente cuando se cambie de pestaña o se cargue una nueva ventana. Si AutoCommit no está activado, se recibirá un mensaje emergente pidiendo guardar o cancelar los cambios.

##### 5.2.1. Propiedades de Campo

Para editar las propiedades de un campo, debe hacerse clic en un campo en el Navegador, el área Registro Único o el área Vista de Cuadrícula. Luego, seleccionar la pestaña **Propiedades de Campo** (*Field Properties*) para ver las propiedades del campo.

Las siguientes propiedades de campo están disponibles para edición o visualización:

- **Nombre** (*Name*): Ingrese la etiqueta o indicación para el campo. Puede cambiarse en el panel Propiedades o escribiendo la etiqueta directamente en el área Registro Único.

- **Descripción** (*Description*): Ingrese una descripción para el campo.

- **Comentario** (*Comment*): Ingrese un comentario para el campo.

- **Mantenido Centralmente** (*Centrally Maintained*): Desmarque esta propiedad para indicar que el Nombre, la Descripción y el Comentario deben recuperarse de los valores establecidos en las propiedades del campo y no del Elemento del Sistema. Esta propiedad se desmarca automáticamente si se actualiza el Nombre, la Descripción o el Comentario.

- **Solo Lectura** (*Read Only*): Marque esta propiedad para indicar que este campo debe ser de solo lectura. Si esta propiedad está marcada, el campo no será editable y se mostrará en gris.

- **Lógica de Visualización** (*Display Logic*): Ingrese una condición que, si es verdadera, da como resultado que el campo se muestre y, si es falsa, da como resultado que el campo se oculte.

- **Longitud de Visualización** (*Display Length*): Seleccione Corto o Largo para controlar la longitud del campo mostrado en la ventana.

- **Cifrado** (*Encrypted*): Marque esta propiedad para indicar que los valores en este campo deben cifrarse.

- **Enfoque Predeterminado** (*Default Focus*): Marque esta propiedad para indicar que el enfoque debe establecerse por defecto en este campo.

- **No. de Orden de Registro** (*Record Sort No.*): Ingrese un número positivo o negativo para indicar que esta columna debe utilizarse para ordenar los datos recuperados en esta ventana.

- **Ocultar** (*Obscure*): Seleccione un tipo de ocultamiento para reemplazar ciertos caracteres en el valor de datos del campo con asteriscos.

- **Sobrescritura de Obligatoriedad** (*Mandatory Overwrite*): Anule la configuración de obligatoriedad para el campo. Si el valor de esta propiedad está en blanco, la propiedad de obligatoriedad se hereda de la columna vinculada al campo. Si el campo es obligatorio, se mostrará en amarillo y necesitará tener un valor, ya sea establecido automáticamente por defecto o ingresado por el usuario, antes de que se pueda guardar un registro.

- **Sobrescritura de Lógica Predeterminada** (*Default Logic Overwrite*): Anule la lógica predeterminada para el campo. Pueden utilizarse literales (por ejemplo, 'Texto' o 123), variables o SQL.

- **Grupo de Campos** (*Field Group*): Este es un campo de solo visualización. Indica si el campo está incluido en un grupo de campos específico.

##### 5.2.2. Propiedades de Columna

Cada campo está vinculado a una columna. Para editar las propiedades de columna, debe seleccionarse la pestaña **Propiedades de Columna** (*Column Properties*).

Las siguientes propiedades de columna están disponibles para edición o visualización:

- **Nombre** (*Name*): Ingrese un nombre para la columna vinculada del campo seleccionado.

- **Descripción** (*Description*): Ingrese una descripción para la columna.

- **Comentario** (*Comment*): Ingrese un comentario para la columna.

- **Tabla** (*Table*): Propiedad de solo lectura que muestra la tabla en la que existe la columna.

- **Nombre de Columna de BD** (*DB Column Name*): Propiedad de solo lectura que muestra el nombre de la columna de base de datos.

- **SQL de Columna** (*Column SQL*): Ingrese un fragmento de SQL en SQL de Columna para columnas virtuales.

- **Table UID**: Utilice la columna como parte de un identificador único para registros en la tabla.

- **Longitud** (*Length*): Ingrese la longitud de la columna en la base de datos.

- **Traducido** (*Translated*): Indique si la columna debe traducirse o no.

- **Referencia** (*Reference*): Propiedad de solo lectura que muestra la referencia de columna (Tabla, Lista, Sí-No, etc.).

- **Lógica Predeterminada** (*Default Logic*): Ingrese lógica para derivar el valor predeterminado para la columna. Pueden ingresarse literales, variables o SQL.

- **Clave** (*Key*): Indique si la columna es una clave en la tabla de base de datos.

- **Columna de Enlace Padre** (*Parent Link Column*): Indique si la columna es un enlace a la tabla padre.

- **Obligatorio** (*Mandatory*): Indique si se requiere un valor para la columna.

- **Obligatorio UI** (*Mandatory UI*): Indique si se requiere un valor para los campos vinculados a la columna.

- **Lógica de Obligatoriedad** (*Mandatory Logic*): Ingrese lógica que determine si se requiere un valor para la columna.

- **Actualizable** (*Updateable*): Indique si los campos vinculados a esta columna pueden actualizarse.

- **Siempre Actualizable** (*Always Updateable*): Indique si la columna siempre es actualizable, incluso si el registro no está activo o procesado.

- **Lógica de Solo Lectura** (*Read Only Logic*): Ingrese lógica que determine si los campos vinculados a la columna serán de solo lectura.

- **Llamada** (*Callout*): Indique si la columna implementa una llamada.

- **Código de Llamada** (*Callout Code*): Ingrese el nombre de clase y método completamente calificado para una llamada en la columna.

- **Identificador** (*Identifier*): Indique si la columna es parte del identificador de registro.

- **Secuencia** (*Sequence*): Indique la secuencia de la columna en el identificador.

- **Selección** (*Selection*): Indique si la columna debe utilizarse como criterio de búsqueda para encontrar registros.

- **Secuencia de Selección** (*Selection Sequence*): Indique la secuencia de la columna en los criterios de búsqueda de selección.

- **FK Recursiva** (*Recursive FK*): Indique si la columna es parte de una clave foránea recursiva.

##### 5.2.3. Agregar un Nuevo Campo

Para agregar un nuevo campo a la ventana:

1. Haga clic en el icono **Nuevo Campo** (*New Field*) en la barra de herramientas  
2. Ingrese un **Nombre de Campo** y seleccione un **Tipo de Entidad**  
3. Ingrese un **Nombre de Columna** al cual se vinculará el campo  
4. Ingrese un **Nombre de Columna de BD** para indicar la columna de base de datos que se creará  
5. Seleccione un **Tipo de Entidad** para la columna  
6. Seleccione una **Referencia** apropiada para la columna (por ejemplo, si se desea agregar un campo de casilla de verificación, seleccione Sí-No; si se desea agregar un campo de botón, seleccione Botón)  
7. Ingrese una **longitud** para la columna de base de datos  
8. Haga clic en **Aceptar**  

Esto creará un nuevo campo y columna. Ahora puede arrastrarse el campo para posicionarlo en el diseño. Seleccione el campo y actualice sus propiedades de campo y columna según sea necesario.

Cuando se guarden los cambios, la columna se creará automáticamente en la base de datos.

### 6. Laboratorio Práctico: Acceso al Entorno de Demonstración

Ahora que hemos explorado la teoría detrás del Diccionario de Datos, es momento de poner manos a la obra. Para que pueda experimentar directamente con las funcionalidades descritas en esta lectura, contamos con un entorno de demostración en línea que le permitirá navegar por el Diccionario de Datos, crear tablas personalizadas y configurar ventanas sin necesidad de instalar nada en su equipo local.

#### 6.1. El Laboratorio: iDempiere 13 Orion

El entorno de prácticas está montado sobre **iDempiere 13 Orion**, la versión más reciente de la plataforma. Se trata de una instalación comunitaria mantenida por GlobalQSS, uno de los contribuyentes más activos del ecosistema iDempiere. A continuación, los detalles técnicos del servidor que estará explorando:

| Componente | Detalle |
|------------|---------|
| **Versión de iDempiere** | 13.0.0.202606091508 |
| **Versión de Base de Datos** | 202605270922_IDEMPIERE-7019.sql |
| **Proveedor** | Soportado por la comunidad iDempiere |
| **Máquina Virtual Java** | OpenJDK 64-Bit Server VM 17.0.19+10-1-22.04.2-Ubuntu |
| **Sistema Operativo** | Linux 5.15.0-185-generic |
| **Servidor Anfitrión** | demo.globalqss.com |

#### 6.2. Cómo Acceder al Laboratorio

El acceso es directo desde cualquier navegador web moderno. Simplemente diríjase a la siguiente dirección:

> 🔗 **https://demo.globalqss.com/webui/**

Al abrir la URL, se le presentará la pantalla de inicio de sesión de iDempiere. Aquí es donde muchos estudiantes se hacen la misma pregunta: *¿qué usuario debo utilizar?* La respuesta depende de lo que quiera hacer. Veamos las opciones disponibles.

##### 6.2.1. Los Cuatro Perfiles de Acceso

El entorno de demostración incluye una empresa de ejemplo llamada **GardenWorld** (un vivero ficticio, clásico en la documentación de Compiere/iDempiere desde sus orígenes). Para interactuar con ella y con el sistema en general, dispone de cuatro cuentas preconfiguradas:

| Usuario | Contraseña | ¿Qué puede hacer? |
|---------|------------|-------------------|
| **GardenAdmin** | GardenAdmin | Administrar la empresa GardenWorld: crear órdenes de venta, gestionar productos, emitir facturas. Es el perfil típico de un usuario funcional con privilegios administrativos sobre la empresa. |
| **GardenUser** | GardenUser | Operar sobre GardenWorld con permisos más limitados. Ideal para simular el rol de un empleado que registra transacciones sin tocar la configuración. |
| **SuperUser** | System | El "comodín" del sistema: puede entrar a cualquier cliente (empresa) y asumir cualquier rol. Es útil para auditoría o para saltar entre contextos, pero no es la opción más limpia para trabajar en el Diccionario de Datos. |
| **System** | System | El administrador técnico del sistema. Es la llave maestra para acceder al **cliente System**, donde residen las definiciones del Diccionario de Datos. |

##### 6.2.2. System o SuperUser para el Diccionario de Datos

Esta es una duda frecuente y vale la pena detenerse en ella. Ambos usuarios tienen privilegios elevados, pero cumplen propósitos distintos dentro de la arquitectura de iDempiere:

- **SuperUser** es un "super-usuario funcional". Puede acceder a cualquier organización o cliente del sistema, pero su naturaleza es transversal: está pensado para supervisión, no para configuración técnica del núcleo.  
- **System**, en cambio, es el usuario nativo del **cliente System**, que es donde iDempiere almacena todas las definiciones del Diccionario de Aplicaciones (*Application Dictionary*): tablas, columnas, ventanas, pestañas, campos, procesos, reglas de validación, elementos y más.  

**Conclusión práctica**: para trabajar con el Diccionario de Datos, inicie sesión con el usuario **System** y la contraseña **System**, seleccionando el rol **System Administrator** y el cliente **System**. Esta combinación le dará acceso limpio y directo a todas las ventanas del menú *Application Dictionary*.

> 💡 **Recomendación didáctica**: si desea comparar cómo se ve una misma entidad (por ejemplo, la tabla `C_Order`) desde la perspectiva funcional y desde la perspectiva técnica, abra dos sesiones del navegador: una como **GardenAdmin** en GardenWorld (para ver la Orden de Venta como usuario) y otra como **System** en el cliente System (para ver la definición de la tabla `C_Order` en el Diccionario de Datos). Esta comparación es extraordinariamente ilustrativa para comprender la Arquitectura Dirigida por Modelos.

##### 6.2.3. Ruta de Navegación hacia el Diccionario de Datos

Una vez dentro del sistema con las credenciales adecuadas, encontrará el Diccionario de Datos organizado dentro del menú principal. La ruta típica es:

```
Menú Principal → System Admin → General Rules → Application Dictionary
```

Desde esta carpeta accederá a las seis entidades fundamentales que estudiamos en esta lectura:

1. **System Element** → para gestionar los Elementos  
2. **Table and Column** → para definir Tablas y Columnas  
3. **Window, Tab & Field** → para construir Ventanas, Pestañas y Campos  
4. **Report and Process** → para configurar Informes y Procesos  
5. **Validation Rule** → para establecer Reglas de Validación  
6. **Context** → para definir Contextos de visualización  

##### 6.2.4. Precauciones Importantes

El entorno de demostración es **compartido** entre múltiples usuarios de la comunidad iDempiere a nivel mundial. Esto implica algunas consideraciones prácticas:

- **No guarde datos sensibles**: cualquier cosa que cree puede ser vista o modificada por otros usuarios.  
- **Los cambios pueden perderse**: periódicamente el servidor se reinicia y la base de datos se restaura a su estado original.  
- **Siga las convenciones de nomenclatura**: si crea tablas o ventanas para practicar, utilice prefijos distintivos (por ejemplo, `XX_MiTabla`) para no colisionar con el trabajo de otros estudiantes.  
- **Aproveche el Editor Visual**: recuerde que puede ejecutar el proceso *Cache Reset* si sus cambios no se reflejan inmediatamente en la interfaz.  

Este laboratorio es su espacio de experimentación. Úselo para replicar los ejemplos de esta lectura, probar las definiciones que se le ocurran y, sobre todo, para comprender cómo el Diccionario de Datos transforma la manera en que se construyen los sistemas empresariales modernos.

### 7. Recursos Adicionales

#### 7.1. Videos Tutoriales

- **iDempiere Visual Dictionary Editor**: Demostración de lo fácil que es personalizar iDempiere. Arrastre y suelte campos fácilmente para crear el diseño de sus ventanas. Defina la lógica de negocio para la validación de datos sin necesidad de programar extensivamente. iDempiere ofrece una experiencia de usuario altamente interactiva, segura y basada en la web, accesible desde cualquier navegador.

- **Application Dictionary Demo with Packout**: Esta demostración muestra cómo crear un nuevo campo con el Diccionario de Aplicaciones y cómo empaquetarlo para su redistribución.

- **The Fastest Way To Build A New ERP**: El desarrollo de aplicaciones ERP ahora es fácil y rápido, con prácticamente ningún problema de programación y mínimas complicaciones. Solo concéntrese en su modelo de datos y en el diseño de su ERD: entradas, procesos y salidas.

#### 7.2. Plugin: RED1 NINJA / iDempiere Ninja

**Generador de Plugins para iDempiere**

Genera plugins completos para iDempiere a partir de definiciones simples en Excel. No se requiere programación para modelos básicos de AD.

**Características:**
- Generación de modelos a partir de definiciones en Excel/CSV  
- Generación automática de tableros Kanban  
- Generación de formatos de impresión  
- Integración con traducción  
- Generación de estructura de flujos de trabajo  
- Operación silenciosa desde la línea de comandos  
- Generación independiente de paquetes 2Pack  
- Todo el esqueleto de plugins OSGI completo  
- Soporte para hojas de Procesos y Menús  

### 8. Conclusiones y Reflexiones Finales

A lo largo de esta lectura hemos recorrido un camino que va desde los fundamentos conceptuales de la Arquitectura Dirigida por Modelos hasta las herramientas prácticas para interactuar con el Diccionario de Datos de iDempiere. Antes de cerrar, conviene detenerse a reflexionar sobre las ideas centrales que atraviesan todo el contenido y sobre las implicaciones más profundas de lo estudiado.

#### 8.1. La Separación entre Modelo y Presentación como Principio Eterno

La idea más poderosa que subyace a todo el sistema iDempiere —y que ha sobrevivido a más de dos décadas de evolución tecnológica— es la **separación radical entre el modelo de datos y su representación en la interfaz de usuario**. Este principio, concebido por Jorg Janke a finales de los años noventa, resulta hoy más vigente que nunca.

En un mundo donde las interfaces cambian constantemente (del cliente Swing al cliente web ZK, de las pantallas de escritorio a las aplicaciones móviles), el modelo de datos permanece como la verdadera esencia del sistema empresarial. Quien domina el modelo, domina la aplicación; quien depende únicamente de la interfaz, queda esclavo de su efímera apariencia. El Diccionario de Datos es, en este sentido, la materialización técnica de una idea filosófica profunda: **los datos son el activo permanente; las pantallas son solo ventanas temporales hacia ellos**.

#### 8.2. La Democratización del Desarrollo: Un Visionario Anticipado

Resulta notable observar cómo la "prueba de los 5 minutos" de Compiere, presentada en 2008, anticipaba en casi una década el movimiento *low-code* y *no-code* que hoy domina la industria del desarrollo de software. Janke y su equipo comprendieron temprano que la personalización empresarial no debía ser patrimonio exclusivo de los programadores.

Esta visión democratizadora encuentra su expresión más acabada en el **Editor Visual del Diccionario de Datos**, donde un usuario funcional puede, mediante gestos intuitivos de arrastrar y soltar, reconfigurar ventanas completas, definir reglas de negocio y agregar nuevos campos. Lo que en otros sistemas requeriría equipos de desarrollo, en iDempiere puede ser realizado por el analista funcional que mejor comprende el proceso empresarial. Esta no es una simple comodidad: es una transformación estructural en la manera en que las organizaciones construyen sus sistemas de información.

#### 8.3. La Resiliencia de las Buenas Ideas Arquitectónicas

La genealogía Compiere → ADempiere → iDempiere nos enseña una lección valiosa sobre la ingeniería de software: **las buenas ideas arquitectónicas sobreviven a las organizaciones que las crean**. Dos *forks* comunitarios, cambios de licencia, migraciones tecnológicas radicales (de JBoss a OSGi, de Java 6 a Java 17), y más de veinte años de evolución no han logrado erosionar el núcleo conceptual del Diccionario de Datos.

Esto debería hacernos reflexionar como futuros arquitectos de software. Cuando diseñamos un sistema, no estamos construyendo solo para el presente: estamos estableciendo principios que, si son suficientemente sólidos, podrán ser reinterpretados y extendidos por generaciones de desarrolladores. La inversión en una abstracción bien pensada —como lo es el Diccionario de Datos— paga dividendos durante décadas.

#### 8.4. El Equilibrio entre Estandarización y Personalización

Uno de los desafíos más sutiles que enfrenta todo sistema empresarial es el equilibrio entre la **estandarización** (que permite mantenimiento, actualización y coherencia) y la **personalización** (que permite adaptación a necesidades específicas). iDempiere resuelve esta tensión mediante un mecanismo elegante: la **centralización con sobrescritura localizada**.

Los Elementos definen las etiquetas de forma centralizada, pero una ventana específica puede sobrescribirlas mediante el Contexto. Las Columnas establecen la lógica predeterminada, pero los Campos pueden modificarla para casos particulares. Las Reglas de Validación se definen una vez y se reutilizan en múltiples lugares. Este diseño permite que el sistema evolucione de manera coherente sin impedir la adaptación local. Es, en esencia, una aplicación del principio de "herencia con sobrescritura" de la programación orientada a objetos, llevado al nivel de la configuración empresarial.

#### 8.5. Hacia una Comprensión Integral del Sistema Empresarial

Al finalizar esta lectura, el estudiante debería haber adquirido no solo conocimientos técnicos sobre cómo configurar tablas, ventanas y campos en iDempiere, sino también una **comprensión integral de por qué el sistema está diseñado de esta manera**. El Diccionario de Datos no es una mera herramienta de configuración: es la manifestación concreta de una filosofía de diseño que prioriza la consistencia, la reutilización y la separación de responsabilidades.

Cuando en el laboratorio práctico acceda al entorno de demostración de iDempiere 13 Orion, no lo haga con la mentalidad de quien completa un ejercicio mecánico. Hágalo con la curiosidad de quien explora una catedral arquitectónica: cada tabla, cada ventana, cada regla de validación es el resultado de más de dos décadas de refinamiento colectivo. Observe cómo las entidades del Diccionario de Datos se relacionan entre sí, cómo los Elementos se reutilizan en múltiples contextos, cómo las Reglas de Validación orquestan el comportamiento dinámico del sistema.

#### 8.6. Una Invitación a la Profundización

Esta lectura constituye solo una puerta de entrada al vasto universo del Diccionario de Datos de iDempiere. Quedan fuera de su alcance temas igualmente fascinantes como los **procesos de empaquetado** (*2Pack*), la **migración entre instancias**, la **integración con flujos de trabajo** (*Workflow*), el **manejo de multitenencia** (*Multi-Tenancy*) y la **extensibilidad mediante plugins OSGi**. Cada uno de estos temas merece una lectura propia y profundiza aún más en las posibilidades que ofrece la Arquitectura Dirigida por Modelos.

La invitación final es clara: **no se conforme con usar iDempiere; comprenda cómo piensa iDempiere**. Quien logra este cambio de perspectiva deja de ser un usuario del sistema para convertirse en un arquitecto de soluciones empresariales. Y es precisamente esa transformación —del operador al arquitecto— el objetivo último de esta lectura y de todo el curso del que forma parte.

---

### 9. Referencias

ADempiere Community. (2006). *From Compiere to ADempiere!* https://www.adempiere.io/community/wiki/preface/adempiere-outpaces-open-source-erp-compiere.html

Compiere. (2008). *Easier application customization with Compiere* [Video]. SourceForge. https://sourceforge.net/p/compiere/news/

GlobalQSS. (s.f.). *iDempiere 13 Orion online demonstration*. https://demo.globalqss.com/webui/

iDempiere Community. (2011). *Founders: Carlos Ruiz and Heng Sin Low*. https://idempiere.org/founders/

iDempiere Wiki. (s.f.). *Application dictionary*. https://wiki.idempiere.org/en/Application_Dictionary

iDempiere Wiki. (s.f.). *I-Dempiere origins: HengSin*. https://wiki.idempiere.org/en/I-Dempiere_Origins:_HengSin

iDempiere Wiki. (s.f.). *Login help*. https://wiki.idempiere.org/en/Login_Help

Janke, J. (2007). *Compiere's origin and history*. Jorg Janke's Blog. http://www.jorgjanke.com/blog/

Low, H. S. (2010). *I-Dempiere origins: HengSin*. iDempiere Wiki. https://wiki.idempiere.org/en/I-Dempiere_Origins:_HengSin

Ruiz, C. (2022, octubre 22). *Historia y arquitectura modular en IDempiere (OSGI)*. iDempiere Perú. https://www.idempiereperu.org/2022/10/22/idempeire-historia-y-arquitectura-modular-en-idempiere-osgi/

Wikipedia. (2024). *Adempiere*. https://en.wikipedia.org/wiki/Adempiere

Wikipedia. (2024). *IDempiere*. https://en.wikipedia.org/wiki/IDempiere

Zavala, J. (s.f.). *Diccionario de datos en Compiere/iDempiere*. Repositorio SiE. https://github.com/jzavalar/SiE/blob/main/06.3_Stack-5_compiere_diccionario-de-datos.md

---

*Documento adaptado para fines educativos sobre Model Driven Architecture en sistemas empresariales modernos.*
