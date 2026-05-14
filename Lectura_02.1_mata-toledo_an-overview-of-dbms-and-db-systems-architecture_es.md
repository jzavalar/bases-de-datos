### **Una visión general de los DBMS y la arquitectura de sistemas de bases de datos**

**Fuente:** Mata-Toledo, R., & Cushman, P. (2000). An overview of DBMS and DB systems architecture. In *Schaum's outline of fundamentals of relational databases* (pp. 1-25). McGraw-Hill.

#### 1.1 Introducción a los Sistemas de Gestión de Bases de Datos

Un ***Sistema de Gestión de Bases de Datos*** (***Database Management System***, ***DBMS***) es el sistema de software que permite a los usuarios definir, crear y mantener una base de datos y proporciona acceso controlado a los datos. Una base de datos es una colección lógicamente coherente de datos con algún significado inherente. El término ***base de datos*** (***database***) se usa frecuentemente para referirse a los datos mismos; sin embargo, existen otros componentes adicionales que también forman parte de un sistema completo de gestión de bases de datos. La figura 1-1 muestra que un DBMS completo usualmente consiste en hardware, software incluyendo utilerías, datos, usuarios y procedimientos. Estos elementos se explicarán en los siguientes párrafos.

[Descripción de imagen: Un círculo etiquetado como "Sistema de Cómputo Completo" que contiene cinco íconos: Hardware (representado por un monitor/terminal) Software (representado por un disco) Datos (representado por un cilindro de base de datos) Usuarios (representado por un ícono de rostro) Procedimientos (representado por una pila de papeles)]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-1.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1-1. Sistema de cómputo completo</div>
</div>


[p. 2]

El ***hardware*** es el sistema de cómputo físico utilizado para almacenar y acceder a la base de datos. En organizaciones grandes, el hardware para dicho sistema típicamente consiste en una red con un servidor central y muchos programas cliente ejecutándose en computadoras de escritorio. El servidor es el procesador central donde la base de datos se encuentra físicamente ubicada. El servidor usualmente tiene un procesador más potente porque maneja las operaciones de recuperación de datos y la mayor parte de la manipulación real de los datos. Los clientes son los programas que interactúan con el DBMS relacional y se ejecutan en las computadoras personales del usuario para acceder a la base de datos. Un DBMS y sus clientes también pueden residir en una sola computadora. En ese caso, usualmente solo hay un usuario accediendo a la base de datos a la vez, ya sea un solo usuario o un sistema de gestión de bases de datos personales accesado por varios usuarios en momentos diferentes. La configuración real de la red varía de organización a organización. Los temas específicos de hardware no se abordarán en este libro.

El ***software*** es el DBMS propiamente dicho. En una red cliente/servidor, el DBMS permite que los programas de manejo de datos residan en el servidor y los programas cliente en cada escritorio. En un sistema de usuario único, usualmente solo una pieza de software maneja todo. El DBMS permite a los usuarios comunicarse con la base de datos. En cierto sentido, actúa como intermediario entre la base de datos y los usuarios. A cada estación cliente o usuario individual se le pueden asignar diferentes niveles de acceso a los datos. Algunos podrán cambiar porciones de la estructura de la base de datos, otros podrán modificar los datos existentes, y otros solo tendrán permiso para visualizar los datos. El DBMS controla el acceso y ayuda a mantener la consistencia de los datos. Las utilerías usualmente se incluyen como parte del DBMS. Algunas de las más comunes son generadores de reportes, herramientas de desarrollo de aplicaciones y otras ayudas de diseño. Ejemplos de software de DBMS incluyen Microsoft Access™, Oracle Corporation Personal Oracle™ e IBM DB2™. La disposición típica de todos los módulos de software y cómo interactúan en un DBMS se explicará al final de este capítulo.

La base de datos debe contener todos los ***datos*** (***data***) necesarios para la organización. Una de las características principales de las bases de datos es que los datos reales están separados de los programas que los utilizan. El conjunto de hechos representados en una base de datos se denomina Universo del Discurso (***Universe of Dicourse***, UOD). El UOD solo debe incluir hechos que formen una colección lógicamente coherente y que sean relevantes para sus usuarios. Por esta razón, una base de datos siempre debe diseñarse, construirse y poblarse para una audiencia particular y con un propósito específico. Probablemente, como parte de la discusión sobre el UOD, es importante señalar que solo una vista parcial del mundo real puede ser capturada por un DBMS. El énfasis está en los datos relevantes pertaining a uno o más objetos, o ***entidades***. Definiremos una entidad como aquello de lo cual se necesita conocer información. Las características que describen o califican una entidad se denominan ***atributos*** de la entidad. Por ejemplo, en una base de datos de estudiantes, la entidad básica es el estudiante. La información registrada sobre esa entidad podría ser nombre y apellido, especialidad, promedio de calificaciones, dirección particular, dirección actual, fecha de nacimiento y nivel académico. Estos son los atributos de la entidad estudiante. El sistema no estaría interesado en el tipo de ropa, el número de amigos, las películas a las que asiste el estudiante, etc. Es decir, esta información no es relevante para el usuario y no debería formar parte del UOD. Se dará más explicación sobre los datos en la siguiente sección.

[p. 3]

Además, para cada atributo, el conjunto de valores posibles que puede tomar se denomina ***dominio*** del atributo. El dominio de la fecha de nacimiento serían todas las fechas que podrían ser razonables en el cuerpo estudiantil; no se esperaría ninguna de los años 1700. Los niveles de clase de licenciatura probablemente estarían restringidos a Freshman (primer año), Sophomore (segundo año), Junior (tercer año) y Senior (cuarto año). No se permitirían otros valores para ese atributo.

Existen diversos tipos de ***usuarios*** que pueden acceder o recuperar datos bajo demanda utilizando las aplicaciones e interfaces proporcionadas por el DBMS. Cada tipo de usuario requiere diferentes capacidades de software.

- El ***administrador de base de datos*** (***database administrator***, DBA) es la persona o grupo encargado de implementar el sistema de base de datos dentro de la organización. El DBA tiene todos los privilegios del sistema permitidos por el DBMS y puede asignar (grant) y revocar (revoke) niveles de acceso (privilegios) a y desde otros usuarios.

- Los ***usuarios finales*** (***end users***) son las personas que se sientan en estaciones de trabajo e interactúan directamente con el sistema. Pueden necesitar responder solicitudes de personas externas a la organización, encontrar respuestas rápidas a preguntas de la alta dirección o generar reportes periódicos. En algunos casos, se debe permitir a los usuarios finales modificar datos dentro del sistema, por ejemplo, direcciones o información de pedidos. Otros usuarios finales, como los del centro de ayuda (help desk), solo necesitarían privilegios para visualizar los datos, no para modificarlos.

- Los ***programadores de aplicaciones*** interactúan con la base de datos de una manera diferente. Acceden a los datos desde programas escritos en lenguajes de alto nivel como Visual Basic o C++. Los programadores de aplicaciones diseñan sistemas como nómina, inventario y facturación, que normalmente necesitan acceder y modificar los datos.

Una parte integral de cualquier sistema es el conjunto de ***procedimientos*** que controlan el comportamiento del sistema; es decir, las prácticas reales que siguen los usuarios para obtener, ingresar, mantener y recuperar los datos. Por ejemplo, en un sistema de nómina, ¿cómo recibe el empleado las horas trabajadas y cómo se ingresan al sistema? ¿Exactamente cuándo se generan los reportes mensuales y a quiénes se envían? Estos procedimientos suelen formalizarse para que los usuarios en cualquier nivel sepan exactamente qué hacer y cómo realizar la tarea asignada. En muchas organizaciones, si algunos empleados han estado ahí por mucho tiempo, pueden saber exactamente qué hacer y cuándo. Sin embargo, es importante tener procedimientos claramente articulados y documentados por escrito para que el sistema no se vea comprometido si nuevos empleados necesitan utilizarlo. Parte del trabajo del DBA es verificar que todos los procedimientos relacionados con el sistema completo estén claramente delineados.

> **Ejemplo 1.1**

> Indique qué tipo de usuario realizaría las siguientes funciones para un sistema de nómina en una empresa grande: (a) Escribir un programa de aplicación para generar e imprimir los cheques, (b) cambiar la dirección en la base de datos de un empleado que se ha mudado, (c) crear una nueva cuenta de usuario para un empleado de nómina recién contratado.

[p. 4]

a.  Escribir un programa de aplicación para generar e imprimir los cheques.

Un programador de aplicaciones o un equipo de programadores diseñaría e implementaría dicho programa de aplicación.

b.  Cambiar la dirección en la base de datos de un empleado que se ha mudado.

Un usuario final podría tomar la información del empleado por teléfono y acceder directamente a la base de datos para cambiarla. Sin embargo, modificar dicha información a partir de conversaciones telefónicas puede resultar en datos incorrectos debido a errores tipográficos o malentendidos. Para verificar que las actualizaciones se realicen correctamente, los procedimientos en muchas organizaciones requieren que los cambios en la base de datos se presenten por escrito.

c.  Crear un nuevo ID de usuario para el empleado de nómina recién contratado.

El DBA o un asistente del DBA que trabaje bajo la supervisión del DBA sería la persona encargada de crear los nuevos IDs de usuario. En una organización pequeña, podría haber solo una persona que realice toda la administración del sistema. En organizaciones más grandes, los asistentes del DBA en el equipo de administración de bases de datos tendrían asignados diferentes trabajos. Una persona podría encargarse de todas las cuentas de usuario y otra podría estar a cargo del mantenimiento de la base de datos.

#### 1.1.1 DATOS

Los datos son el corazón del DBMS. Existen dos tipos de datos. Primero, y el más obvio, es la colección de información necesaria para la organización. El segundo tipo de datos, o ***metadatos***, es información sobre la base de datos. Esta información usualmente se guarda en un ***diccionario de datos*** o ***catálogo***. El diccionario de datos incluye información sobre usuarios, privilegios y la estructura interna de la base de datos. Una gestión cuidadosa de todos los datos es esencial para que la información pueda considerarse actualizada y precisa. Todos los niveles de usuarios necesitan tener un entendimiento firme de la base de datos y de cómo está estructurada. Es útil examinar la base de datos desde varias perspectivas diferentes. El sistema puede ser multiusuario o de usuario único; los datos usualmente están tanto integrados como compartidos; y la base de datos puede estar centralizada o distribuida.

Primero, la configuración del hardware y el tamaño de la organización determinarán si se trata de un sistema ***multiusuario*** (***multi-user system***) o un ***sistema de usuario único*** (***single-user system***). En un sistema de usuario único, la base de datos reside en una sola computadora y solo es accesada por un usuario a la vez. Este único usuario puede diseñar, mantener y escribir programas para el sistema, desempeñando todos los roles de usuario. Por otro lado, podría haberse contratado a otra persona, frecuentemente un consultor, para diseñar el sistema. En este caso, el usuario único podría solo desempeñar el rol de usuario final y los datos podrían siempre ser accesados interactivamente a través del DBMS sin utilizar programas de aplicación.

Debido a la gran cantidad de datos gestionados incluso por organizaciones pequeñas, la mayoría de los sistemas son multiusuario. [p. 5] En esta situación, los datos están tanto ***integrados*** como ***compartidos***. Una base de datos está integrada cuando la misma información no se registra en dos lugares. Por ejemplo, tanto el departamento de facturación como el de envíos pueden necesitar las direcciones de los clientes. Aunque ambos departamentos puedan acceder a diferentes porciones de la base de datos, las direcciones de los clientes deben residir en un solo lugar. Es trabajo del DBA asegurarse de que el DBMS haga disponibles las direcciones correctas desde un área de almacenamiento central.

Asimismo, las piezas individuales de datos son compartidas por ambos departamentos. El DBMS debe asegurar que los dos usuarios no estén modificando diferentes porciones de los datos al mismo tiempo. Si esto ocurre, los datos podrían no permanecer precisos. Además, los usuarios que comparten datos no necesitan el mismo nivel de acceso. El departamento de envíos solo podría necesitar examinar la dirección del cliente con fines de envío y no debería tener necesidad de examinar el historial de pagos del cliente. El departamento de facturación necesita poder examinar el saldo actual y modificar el saldo cuando se realiza un pago. Estos permisos se denominan ***privilegios*** y, como se indicó anteriormente, son asignados por el DBA.

> **Ejemplo 1.2**

> Considere una base de datos en una empresa de cable que contiene nombres de clientes, direcciones, categorías de servicio (cable básico, canales premium, pago por evento, etc.) e información de facturación. Indique, para cada usuario, un empleado de facturación, un técnico de reparaciones y un representante de servicio al cliente, a qué elementos debería poder acceder y cuáles debería poder modificar.
>
> | Usuario | Nivel de Permiso |
> |---------------------------|---------------------------------------------|
> | a\. Empleado de facturación | Debería poder acceder y modificar todos los datos. |
> | b\. Técnico de reparaciones | Necesita acceder, pero no modificar, nombre, dirección e información de servicio. No debería tener acceso a ninguna información de facturación. |
> | c\. Representante de servicio al cliente | Necesita poder acceder y modificar nombre, dirección e información de servicio. Si las preguntas de facturación se refieren al departamento de facturación, el representante de servicio al cliente no necesita ninguna información de facturación. |

Un tercer aspecto para entender tanto los datos como el DBMS es si el sistema es ***centralizado*** o ***distribuido***. Durante las décadas de 1970 y 1980, la mayoría de los sistemas de gestión de bases de datos residían en grandes *mainframes* o minicomputadoras. Los sistemas eran centralizados y de una ***sola capa*** (***single tier***), lo que significa que el DBMS y los datos residían en una sola ubicación. La teoría sostenía que, si los datos se mantienen en dos lugares, existe una alta probabilidad de que dos elementos que se supone son idénticos en realidad no lo sean. Por ejemplo, si la dirección de un cliente se almacena en dos tablas separadas por alguna razón, es posible que una se modifique y la otra permanezca igual. Frecuentemente se utilizaban terminales tontas (dumb terminals) para acceder al DBMS mediante teleproceso.

[p. 6]

El auge de las computadoras personales en los negocios durante la década de 1980, el aumento de la confiabilidad del hardware de redes y los avances en la realización de negocios a través de Internet durante la década de 1990 llevaron a la nueva tendencia de intentar mantener la precisión de los datos y, al mismo tiempo, hacer uso de sistemas distribuidos. Los sistemas de ***dos capas*** (***two-tier***) y ***tres capas*** (***three-tier***) se volvieron comunes. En un sistema de dos capas, se requiere software diferente para el servidor y para el cliente. El sistema de tres capas añade middleware, que proporciona una forma para que los clientes de un DBMS accedan a datos desde otro DBMS. La figura 1-2 ilustra la diferencia entre configuraciones de software de una, dos y tres capas.

[Ilustración de la Figura 1-2: a. Una sola capa: Un gran bloque rectangular que representa la máquina principal contiene "Software del DBMS" y "Datos". Flechas apuntan desde este bloque a cuatro bloques cuadrados más pequeños que representan terminales. Leyenda: "a. Una sola capa: Todo en una máquina. Accesado por terminales". b. Dos capas: Un bloque rectangular central etiquetado "Software del Servidor del DBMS" y "Datos" está conectado mediante flechas a dos bloques más pequeños etiquetados "Software del Cliente del DBMS". Leyenda: "b. Dos capas: Servidor con muchos Clientes". c. Tres capas: Dos grandes bloques etiquetados "Software del Servidor del DBMS I/Datos" y "Software del Servidor del DBMS II/Datos" están conectados en el medio por un círculo etiquetado "Middleware". Los clientes de cada DBMS se conectan a sus respectivos servidores. Leyenda: "c. Tres capas: Dos tipos de DBMS conectados por Middleware".]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-2.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1-2. Configuraciones de una, dos y tres capas.</div>
</div>

Un DBMS distribuido puede implementarse de varias maneras diferentes. Una red de oficina local podría almacenar los datos de clientes en un servidor y los datos de proveedores en otro. En ambos casos, el sistema permitiría que muchos programas cliente accedieran a los datos de ambos servidores al mismo tiempo. El capítulo 6 analiza las implicaciones de seguridad de esta configuración. [p. 7] Otro método de distribución es almacenar varias bases de datos equivalentes en diferentes lugares. Los datos se distribuyen geográficamente y se ubican lo más cerca posible de donde se utilizarán. Sin embargo, es crítico en una base de datos distribuida que cada nodo del sistema distribuido pueda ejecutar una aplicación global o acceder a archivos en cualquier otro nodo. Por ejemplo, una organización con sucursales en varios estados podría almacenar una lista de clientes diferente en cada sucursal. Las tablas están distribuidas pero conectadas, por lo que el DBMS puede encontrar la información de cualquier cliente en cualquier momento desde cualquier ubicación. Los usuarios solicitan información particular y el DBMS oculta los detalles de cómo localiza los datos solicitados. Esta transparencia es un tema importante en el software de DBMS distribuido. Recuerde, aunque la base de datos puede estar distribuida, no es lo mismo que estar descentralizada. Los elementos de datos siguen residiendo en un solo lugar y el DBMS sabe dónde encontrarlos. Otra ventaja del modelo distribuido es que resulta en una mayor confiabilidad y rendimiento. Cuando tanto los datos como el software del DBMS están dispersos, si un sistema falla, otros deberían poder seguir funcionando y toda la organización no queda inmovilizada.

Existen varias disposiciones posibles para conectar los nodos de un sistema distribuido. Pueden conectarse en una configuración de estrella, anillo o red, como se muestra en la figura 1-3. La configuración en estrella está centralizada y depende del nodo central para la comunicación entre todos los nodos. Si el servidor central tiene problemas, el resto de los nodos son inaccesibles. Tanto en la configuración de anillo como en la de red, la estabilidad de la red no depende completamente de la estabilidad de una sola máquina. Los temas particulares de redes están más allá del alcance de este libro.

[Ilustración de la Figura 1-3: Estrella: Una caja central etiquetada "Servidor Central" está conectada a cinco cajas periféricas mediante líneas. Anillo: Seis cajas están conectadas en un círculo, con flechas que indican un bucle. Red: Una malla de cajas conectadas por múltiples líneas que se intersectan, mostrando varios caminos entre nodos.]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-3.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1-3. Configuraciones de red.</div>
</div>

[p. 8]

> **Ejemplo 1.3**
>
> Describa cómo un grupo médico local de 3 doctores con 2 ubicaciones de oficina separadas mantendría su base de datos de pacientes si fuera centralizada. ¿Cómo podría cambiarse para convertirlo en un sistema distribuido?

Para un sistema centralizado, toda la base de datos residiría en un servidor en una ubicación central. Las máquinas cliente en cada ubicación accederían a la base de datos central para obtener información de pacientes. Si el sistema central estuviera caído, no se podría acceder a ninguna información de pacientes. En un sistema distribuido, la información de los pacientes usualmente vistos en la ubicación 1 se mantendría en un servidor en esa oficina. La información de pacientes de la ubicación 2 se mantendría en un servidor en esa oficina. El DBMS accedería a ambas ubicaciones para encontrar información sobre cualquier paciente. Si uno de los servidores estuviera caído, el otro aún podría ser accesado.

> **Ejemplo 1.4**
>
> Especifique si cada sistema sería de una, dos o tres capas.
>
> a.  La cadena de moteles Happy Nights permite a los gerentes locales comprar una franquicia. Pueden instalar y usar el DBMS de su elección para su sistema de reservaciones. El único requisito es que puedan conectarse y comunicarse con el sistema de la oficina central.
>
> b.  La empresa Sticky Wicket tiene oficinas centrales en Detroit y sucursales en Chicago y Baltimore. La base de datos de inventario y partes está distribuida, y cada sucursal mantiene su propio inventario. Un DBMS central ubicado en Detroit permite pedidos instantáneos de suministros a través de la oficina central.

Dado que ambas empresas tienen varias ubicaciones, obviamente los sistemas no son de una sola capa. La clave de la diferencia es si existe un DBMS central o si cada entidad local ejecuta su propio sistema de base de datos personal. Dado que el ejemplo (a) permite a cada franquicia usar un DBMS diferente, se requerirá middleware para conectar estos sistemas con el DBMS de la oficina central, resultando en un sistema de tres capas. Debido a que (b) utiliza un DBMS central con software cliente en cada ubicación, este sería un sistema de dos capas.

#### 1.1.2 POR QUÉ NECESITAMOS DBMS

Antes de continuar con esta introducción a los conceptos de DBMS, es importante especificar por qué necesitamos sistemas de gestión de bases de datos. Ciertamente, todos los lectores son conscientes de la explosión de información en la sociedad actual. La información personal se almacena sobre cada uno de nosotros en una variedad de formas. Cualquiera que trabaje en cualquier tipo de negocio, ya sea una organización grande o pequeña, sabe lo importante que es mantener registros precisos. Las ventajas de usar un DBMS caen en tres categorías principales:

- Mantenimiento adecuado de los datos

- Proporcionar acceso a los datos

- Mantener la seguridad de los datos

[p. 9]

#### 1.1.2.1 Mantenimiento Adecuado de los Datos

El mantenimiento adecuado de los datos será un tema recurrente a lo largo de este libro. Los usuarios deben poder confiar en que los datos son precisos y están actualizados. Debe evitarse la ***inconsistencia*** y minimizarse la ***redundancia***. La redundancia ocurre cuando la misma información se guarda en varios lugares. La inconsistencia surge cuando los datos se modifican en una de esas ubicaciones y no en otra. La mayoría de los sistemas de bases de datos proporcionan ***restricciones de integridad*** (***integrity constraints***) que deben seguirse. Todos estos conceptos se considerarán tanto explícita como implícitamente en los siguientes capítulos. El DBMS es la clave para hacer cumplir estas características dentro de la base de datos. Cada DBMS puede gestionar los datos de diferentes maneras, pero todos se esfuerzan por abordar estos temas de datos.

> **Ejemplo 1.5**
>
> En una organización particular, los nombres y direcciones de los clientes se mantienen en una base de datos para el departamento de ventas y en otra base de datos para el departamento de facturación. ¿Qué inconsistencia podría resultar de esta redundancia?

Cuando un vendedor está tomando un pedido, el cliente reporta un cambio de dirección. El vendedor podría actualizar el registro en el departamento de ventas. Sin embargo, cuando se prepara la factura, se envía a la dirección anterior porque la dirección no se cambió en esa base de datos.

#### 1.1.2.2 Proporcionar Acceso a los Datos

Como se especificó en la sección anterior, los datos usualmente son compartidos por una variedad de usuarios y programas. Tanto el almacenamiento como el acceso a los datos deben ser fáciles y rápidos. El DBMS debe proporcionar soporte concurrente para todo tipo de transacciones, tanto consultas interactivas como programadas. Las consultas interactivas no deberían tener que esperar a que terminen los programas de aplicación. Básicamente, los datos deben ser accesibles precisamente cuando se requieran. Sería inaceptable que los usuarios tuvieran que esperar incluso un día mientras la base de datos se actualiza y verifica. El trabajo del DBMS es permitir un acceso rápido para todos los usuarios necesarios mientras se siguen utilizando procedimientos de mantenimiento adecuados.

Otro tema relacionado con el acceso es la capacidad de encontrar una pieza particular de información dentro de la gran cantidad de datos almacenados. El DBMS debe contener métodos flexibles para acceder a cada elemento en la base de datos mientras permite búsquedas rápidas a través de toda la base de datos para encontrar ese elemento.

#### 1.1.2.3 Mantener la Seguridad de los Datos

El DBA usualmente es la persona responsable de la seguridad de los datos. Debe prevenirse el acceso no autorizado, y se deben otorgar una variedad de niveles de permiso a los usuarios. Se proporcionan herramientas para que el DBA haga cumplir todos los procedimientos de seguridad y cumpla con todos los requisitos conflictivos que surgen cuando muchas personas deben acceder a la misma base de datos. [p. 10] Si dos usuarios separados están accediendo a una tabla particular al mismo tiempo, el DBMS no debe permitir que ambos realicen cambios conflictivos. Tales salvaguardas son parte de todos los sistemas. El DBMS también proporciona las herramientas para realizar copias de seguridad (backups) y recuperación fáciles en caso de fallas del sistema. El capítulo 6 explora una variedad de temas relacionados con la seguridad de los datos.

> **Ejemplo 1.6**
>
> Haga una lista de todas las bases de datos en las que pueda pensar donde se guarden su nombre e información financiera. ¿Cómo puede verificar la precisión de estos datos?

La información sobre usted es almacenada por su empleador, su escuela, posiblemente su organización religiosa, el gobierno y cualquier banco o compañía de crédito. Puede verificar la precisión de muchas de estas ubicaciones solicitando un reporte de crédito o un estado de cuenta actual. Muchas localidades tienen leyes que requieren que las escuelas y agencias gubernamentales le permitan ver y corregir información personal. Nunca sabrá si la información está equivocada si no la verifica usted mismo.

También podría ser útil especificar situaciones en las que uno podría no querer usar un DBMS. A veces, estos sistemas resultan en costos innecesarios cuando el procesamiento tradicional de archivos funcionaría igual de bien. Si solo una persona mantiene los datos y esa persona no tiene habilidades para diseñar una base de datos, el producto resultante también podría tomar más tiempo y ser menos eficiente. Cuando esté diseñando una base de datos, como con cualquier otro sistema, recuerde que una estrategia simple y clara usualmente se mantiene más fácilmente que un diseño complejo y confuso.

#### 1.2 Modelos de Datos

Los niños construyen modelos de aviones o autos como una forma de entender cómo se construye un avión o auto real. A través de la comprensión de los modelos, los niños esperan ver un auto con cuatro ruedas y un avión con dos alas. Los desarrolladores de bienes raíces llevan a clientes potenciales en recorridos por una casa modelo para mostrarles cómo está organizada la casa y la relación entre las diversas habitaciones. Los compradores pueden hacerse una buena idea del flujo de la cocina al comedor y a la sala. Los modelos generalmente permiten a las personas conceptualizar una idea abstracta más fácilmente.

Un ***modelo de datos*** (***data model***) es una forma de explicar la disposición lógica de los datos y la relación de varias partes entre sí y con el todo. Se han utilizado diferentes modelos de datos a lo largo de los años. En los primeros años, frecuentemente un sistema de ***archivo plano*** (***flat file***), o un archivo de texto simple con todos los datos listados en algún orden, parecía lo más fácil. El programa de aplicación accedía a los datos usualmente de forma secuencial para *procesamiento por lotes* (***batch processing***). No había mucho acceso interactivo disponible. Otros modelos utilizados en grandes mainframes eran el ***jerárquico*** y el ***de red***. La base de datos jerárquica se construye utilizando un modelo de árbol, con una raíz y varios niveles de subárboles. Cada elemento tiene solo un enlace que conduce a él. Los datos se acceden comenzando desde la raíz y descendiendo por el árbol hasta localizar los detalles deseados. El modelo de red contiene muchos enlaces entre los diversos elementos de datos. Los índices interrelacionados permiten acceder a los datos desde una variedad de direcciones.

[p. 11]

En 1970, el Dr. E. F. Codd describió un nuevo tipo de modelo, el ***modelo relacional*** para sistemas de bases de datos.[^1] Los sistemas de gestión de bases de datos relacionales (RDBMS), donde todos los datos se guardan en tablas o relaciones, se convirtieron en el nuevo estándar. Son mucho más flexibles y fáciles de usar, ya que casi cualquier elemento de datos puede ser accesado más rápidamente que en los otros modelos. El tiempo de recuperación se reduce, por lo que el acceso interactivo se vuelve más factible que en los otros modelos. El uso de tablas relacionadas y vistas también permite el uso de bases de datos distribuidas que serían difíciles en los modelos jerárquico o de red. El diccionario de datos para el modelo relacional contiene los nombres de las tablas, junto con los nombres de las columnas y los tipos de datos para cada tabla. Además, el diccionario de datos mantiene información sobre todos los usuarios y privilegios. Los sistemas de gestión de bases de datos relacionales (RDBMS) serán el modelo utilizado a lo largo de este libro y se explican más completamente en el Capítulo 2.

> **Ejemplo 1.7**
>
> Dado este modelo de datos para una empresa de flores, ¿sería jerárquico, de red o relacional?
>
> [Interpretación de imagen: Un diagrama de árbol jerárquico. El nodo raíz está etiquetado como "NOMBRE DE LA FLOR". Se ramifica hacia abajo a tres nodos hijos: "INSTRUCCIONES DE SIEMBRA", "CONDICIONES" y "COSTO". El nodo "CONDICIONES" se ramifica aún más hacia abajo en dos nodos hijos: "NECESIDADES DE LUZ" y "CONDICIONES DEL SUELO".]
>
> Figura ej-1.7

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-4_ej-1-7.png" width="60%">
  <div style="font-size: 0.9em; margin-top: 0.5em;"></div>
</div>

Este sería un modelo jerárquico porque, aunque cada nodo puede apuntar a varios otros nodos, cada nivel solo tiene un nodo apuntando a él desde arriba. Para averiguar la cantidad de luz necesaria, uno necesitaría acceder primero al nombre de la flor y luego a las condiciones. Sería difícil acceder a las instrucciones de siembra solo para aquellas flores que necesitan plantarse a pleno sol.

> **Ejemplo 1.8**
>
> ¿Cómo se organizarían los mismos datos en un RDBMS?

En una base de datos relacional, los datos se guardarían en una tabla con una fila para cada elemento, como se muestra a continuación. Las preguntas respecto a los valores en cualquier columna son posibles en cualquier momento. Por ejemplo, los nombres e instrucciones de siembra para cada flor que crece a pleno sol se mostrarían fácilmente.

[p. 12]

| Flower Name | Planting Instruccions | Conditions | Light Needs | Soil Conditions | Cost |
|------------|------------|------------|------------|------------|------------|
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |

#### 1.3 Arquitectura del Sistema de Bases de Datos

Comprender un modelo abstracto de los datos es importante para poder describir la arquitectura del sistema de bases de datos. Recuerda que, anteriormente en este capítulo, una de las principales características de las bases de datos es que los datos reales están separados de los programas que los utilizan. Ese hecho es importante tenerlo en cuenta en toda esta sección, que explica los esquemas y lenguajes de bases de datos y luego describe la arquitectura del sistema de bases de datos.

Los sistemas de gestión de bases de datos también pueden clasificarse según la forma en que utilizan su diccionario de datos. Como se mencionó anteriormente, el diccionario de datos contiene descripciones lógicas de los datos y sus relaciones, información física sobre el almacenamiento de datos y, generalmente, información sobre los usuarios y privilegios. Algunos proveedores de software también diseñan el diccionario de datos para guardar información de uso, como frecuencias de consultas y otra información de transacciones. Los diccionarios de datos son útiles para todos los usuarios humanos, especialmente el administrador de la base de datos, así como invaluables para los programas de aplicación y los generadores de informes que puedan acceder a la base de datos.

#### 1.3.1 ESQUEMAS Y LENGUAJES

El modelo de datos describe los datos y las relaciones a nivel abstracto. El ***esquema de la base de datos*** (***database schema***) se utiliza para describir la organización conceptual del sistema de bases de datos. Esta organización se define durante el proceso de diseño, generalmente utilizando el ***lenguaje de definición de datos*** (***Data Definition Language***, ***DDL***) proporcionado por el proveedor de software en particular.

La organización de los datos puede definirse en dos niveles: lógico y físico. La organización física está relacionada con cómo se almacenan realmente los datos en el disco. La organización lógica es el modelo de datos conceptual que se está implementando. El DDL permite al usuario definir la organización de los datos a nivel lógico. Luego, el software específico del sistema de gestión de bases de datos (DBMS) se encarga de la organización física de los datos mapeando del nivel lógico al nivel físico. De esta manera, los usuarios están protegidos de tener que lidiar con el almacenamiento a nivel de hardware de los datos.

El DDL se utiliza para crear las tablas y describir los campos dentro de cada tabla. La figura 1-4 muestra el diagrama de esquema para parte de una base de datos de una tienda minorista. El esquema muestra tres tablas. Cada tabla contiene información sobre objetos particulares: clientes, pedidos y empleados. El esquema no contiene información sobre cómo se organizan físicamente los bits de cada elemento ni exactamente dónde se almacena el elemento en un dispositivo de almacenamiento particular.

[p. 13]

| ORDER INFORMATION |  CUSTOMER INFORMATION | EMPLOYEE INFORMATION |
|-------------------|-----------------------|----------------------|
| ID                | ID                    | ID                   |
| CUSTOMER_ID       | NAME                  | LAST_NAME            |
| DATE_ORDERED      | PHONE                 | FIRST_NAME           |
| DATE_SHIPPED      | ADDRESS               | USERID               |
| SALES_REP_ID      | CITY                  | START_DATE           |
| TOTAL             | STATE                 | COMMENTS             |
| PAYMENT_TYPE      | COUNTRY               | MANAGER_ID           |
| ORDER_FILLED      | ZIP_CODE              | TITLE                |
|                   | CREDIT_RATING         | DEPT_ID              |
|                   | SALES_REP_ID          | SALARY               |
|                   | REGION_ID             | COMMISSION_PCT       |

**Fig. 1-4.** Diagrama de esquema de una parte de la base de datos de una tienda minorista.

Es importante decidir el esquema temprano en el proceso de diseño de la base de datos. Una vez que la base de datos ha sido creada y ha comenzado a llenarse con datos, a veces es difícil cambiar el esquema. La población real de la base de datos con información se realiza utilizando el lenguaje de manipulación de datos (*DML*). El lenguaje más común utilizado por muchos sistemas de gestión de bases de datos para este propósito es SQL, que se explica en el Capítulo 3. El DML permite al usuario ingresar, recuperar y actualizar los datos. Algunos sistemas de gestión de bases de datos, como Microsoft Access™, permiten una interacción gráfica con la base de datos, pero generalmente un DML trabaja en segundo plano.

> **Ejemplo 1.9**\
> Diseña un esquema posible para una oficina de médicos. Los doctores quieren acceso inmediato a la información médica de los pacientes. El empleado de registros necesita asegurarse de que todas las aseguradoras sean facturadas y, luego, que cada paciente sea facturado por el resto.

Un esquema simple posible se muestra a continuación. La información podría mantenerse en tres tablas: historial médico del paciente, información personal del paciente y datos de la aseguradora. La información del historial médico sería determinada por la especialidad particular de los doctores. Esta tabla estaría conectada a la información personal a través del patient_ID. La información de la aseguradora podría mantenerse en otra tabla conectada a cada paciente mediante el company_ID. El lector debe ser consciente de que este ejemplo no es la definición de una base de datos completa. Seguramente, se almacenaría más información.

[p. 14]

| **PATIENT MEDICAL HISTORY** | **PATIENT PERSONAL INFORMATION** | **INSURANCE COMPANY INFORMATION** |
|------------------------|------------------------|------------------------|
| PATIENT_ID | PATIENT_ID | COMPANY_ID |
| AGE | NAME | NAME |
| GENDER | ADDRESS | ADDRESS |
| PAST_ILLNESS | CITY | CITY |
| OTHER... | STATE | STATE |
|  | ZIP | ZIP |
|  | TOTAL_AMT_DUE |  |
|  | INSURANCE_CO_ID  |  |
|  |  AMT_BILLED_INSURANCE |  |

> **Ejemplo 1.10**
>
> ¿El usuario utilizaría el DML o el DDL para cada tarea? (a) Cambiar la dirección del cliente, (b) definir una tabla de inventario, (c) ingresar la información de un nuevo empleado.
>
> a.  y **c.** Actualizar la dirección de un cliente y ingresar la información de un nuevo empleado se realizarían mediante el uso del DML. Ambas actividades implican manipular datos dentro de las tablas ya establecidas.
>
> b.  La definición de una nueva tabla implicaría el uso del DDL. Crear la tabla y establecer los atributos son parte de la definición de datos.

#### 1.3.2 ARQUITECTURA DE TRES NIVELES

El método generalmente aceptado para explicar la arquitectura de un sistema de base de datos fue formalizado por un comité en 1975 y explicado con mayor detalle en 1978.[^2]

[p. 15]

Se conoce como arquitectura ANSI/SPARC, nombrada por el Comité de Planificación de Estándares y Requisitos (Standards Planning and Requirements Committee) del Instituto Nacional Estadounidense de Estándares (American National Standards Institute). Los tres niveles son interno, conceptual y externo.

- El ***nivel interno*** es el que se refiere a la forma en que los datos se almacenan físicamente en el hardware. El nivel interno se describe utilizando los bytes reales y la terminología a nivel de máquina. Usualmente, el software del DBMS se encarga de este nivel.

- El ***nivel conceptual***, la definición lógica de la base de datos, a veces se denomina vista de la comunidad. El modelo de datos y el diagrama de esquema son explicaciones de la base de datos en el nivel conceptual. El DBA y sus asistentes mantienen el esquema y usualmente son quienes usan el DDL para definir la base de datos.

- El ***nivel externo*** es el que se refiere a los usuarios. Ya sean programadores de aplicaciones o usuarios finales, aún tienen una vista, o modelo mental, de la base de datos y lo que contiene.

La figura 1-5 muestra una representación gráfica de los tres niveles. El nivel conceptual, o vista de la comunidad, es donde el nivel físico interno se interpreta y se transforma en vistas externas para los usuarios. Esta figura demuestra que el DBMS actúa como el "intermediario" para gestionar el sistema manejando el almacenamiento interno y protegiendo a los usuarios de los problemas de hardware.

[Interpretación de imagen: Un diagrama que ilustra la arquitectura de tres niveles del DBMS. En la parte inferior está "Hardware" conectado al "Nivel Interno". El Nivel Interno conecta hacia arriba al "Nivel Conceptual", que se describe como manejando "mostrando vistas al nivel externo" y "mapeando definición al nivel interno". El Nivel Conceptual conecta hacia arriba al "Nivel Externo". Desde el Nivel Externo, flechas apuntan a varios íconos de usuarios: "Usuario Final 1", "Usuario Final n", "Programador de Aplicaciones 1" y "Programador de Aplicaciones n". Una etiqueta indica "Vistas diferentes para diferentes usuarios".]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-5.png" width="80%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1-5. Arquitectura de tres niveles del DBMS.</div>
</div>

[p. 16]

Para entender la diferencia entre los tres niveles, considere nuevamente el esquema de base de datos en la figura 1-4 que describe clientes, pedidos y empleados. Ese esquema sería la vista conceptual o de la comunidad de la base de datos. Se lista información particular para cada entidad. El nivel interno describiría exactamente qué bytes contienen la información y cómo puede ser accesada. Si el Usuario 1 es el empleado de nómina, la vista externa contendría solo la información de empleados. Si el Programador de Aplicaciones 1 está diseñando programas de facturación, él o ella necesitaría toda la información de clientes y pedidos, así como información sobre el representante de ventas particular en la vista externa. La figura 1-6 muestra información específica realmente disponible en cada nivel respecto a un empleado particular.

[Descripción de la Figura 1-6: Un diagrama que muestra la jerarquía de niveles de base de datos. Arriba: "Nivel Externo" conectado a "Usuario Final 1" (quien dice "Mantengo información sobre el empleado Alex Bell") y "Programador de Aplicaciones 1" (quien dice "Mi programa puede procesar pedidos de los clientes de Alex Bell"). Medio: "Nivel Conceptual" conectado a una caja que muestra "ID Empleado Dept ... Alex Bell 19 43 ...". Abajo: "Nivel Interno" conectado a "Hardware" (una caja) y una caja que muestra "BYTES = 20 FF52 BYTE 1 FF68 BYTE 2".]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-6.png" width="80%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1-6. Información en diferentes niveles.</div>
</div>

> **Ejemplo 1.11**
>
> Examine el esquema de muestra en el Ejemplo 1.9. ¿Cuál sería la vista interna, la vista de la comunidad y la vista externa de esa base de datos?

Las tablas y el contenido real de cada columna serían la vista de la comunidad de la base de datos. Podría contener información como:

| **PATIENT_ID**  | **NAME**  | **ADDRESS...** |
|-------------|-----------|----------------|
| 444-44-4444 | Sue Jones | 345 High St... |

[p. 17]

La vista interna sería la ubicación física de cada elemento en el disco del servidor, así como cuántos bytes de almacenamiento necesita cada elemento. La vista externa dependería de qué usuario esté accediendo a la base de datos. El doctor esperaría ver el historial del paciente. El empleado de facturación esperaría la información de seguros y facturación.

#### 1.3.3 INDEPENDENCIA DE DATOS

El concepto de ***independencia de datos*** (***data independence***) es importante abordar en este momento. Ya se ha dicho que los datos deben mantenerse separados del DBMS. A nivel físico, los datos deben ser independientes del modelo o arquitectura particular. El esquema en cualquiera de los tres niveles debe ser modificable sin interferir con el siguiente nivel superior. Por ejemplo, el almacenamiento físico de la base de datos podría necesitar cambiarse. Sin embargo, este cambio no debería afectar ni la vista conceptual de lo que está almacenado ni la capacidad del usuario para entender y acceder a los datos. Los datos también deben ser lógicamente independientes. Diferentes usuarios y programas de aplicación requieren información diferente a través de diferentes vistas lógicas. Un sistema bien diseñado mantendrá la independencia de datos tanto física como lógicamente.

#### 1.3.4 INTEGRANDO LOS MÓDULOS

Hemos discutido todos los componentes del DBMS. Esta sección se enfoca en los diversos módulos de software que usualmente se encuentran en el DBMS y dónde pueden ubicarse con respecto al sistema de cómputo en su conjunto. Probablemente sea más fácil abordar este entorno desde el punto de vista de los diferentes usuarios explicados anteriormente. La figura 1-7 muestra la relación de los usuarios y los diversos módulos de software con los datos reales.

[Descripción de la Figura 1-7: Un diagrama de flujo titulado "Módulos del DBMS" dividido en Usuarios, Software y Datos. Usuarios: "DBA y Asistentes" -\> "DDL" y "Comandos Privilegiados" "Usuarios Finales" -\> "Consulta Interactiva" "Programadores de Aplicaciones" -\> "Programas de Aplicación"

Software: "DDL" -\> "Compilador DDL" -\> "Sistema y Diccionario de Datos" "Comandos Privilegiados" -\> "Gestor de Datos Almacenados" "Consulta Interactiva" -\> "Compilador de Consultas" -\> "Sistema y Diccionario de Datos" -\> "Procesador de BD en Tiempo de Ejecución" "Programas de Aplicación" -\> "Pre-Compilador" y "Compilador de Lenguaje Huésped" -\> "DML" -\> "Compilador DML" -\> "Sistema y Diccionario de Datos"

Datos: "Sistema y Diccionario de Datos" -\> "Gestor de Datos Almacenados" "Gestor de Datos Almacenados" -\> "Base de Datos Almacenada"]

Fig. 1-7. Módulos del DBMS.
<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-7.png" width="80%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 1-7. Flujo de arquitectura de PostgreSQL</div>
</div>

[p. 18]

Los datos reales se almacenan en un disco. El DBA y sus asistentes pueden emitir tanto comandos privilegiados como declaraciones DDL, que son gestionados primero por el compilador DDL. Los usuarios finales emiten consultas interactivas que también se compilan antes de ser procesadas. Los programadores de aplicaciones escriben los programas que son precompilados para crear las declaraciones DML necesarias, así como compilados en el lenguaje huésped.

El DBMS proporciona protección completa para los datos mediante el uso del diccionario de datos, el gestor de base de datos en tiempo de ejecución y el gestor de datos almacenados. Cada acceso a los datos almacenados pasa por uno o más de estos componentes. El DBMS verifica consistentemente con el diccionario de datos para asegurar que todos los accesos sean legales y luego procesa esos comandos a través del procesador en tiempo de ejecución. El gestor de base de datos en tiempo de ejecución maneja cada consulta, ya sea de recuperación o actualización. Solo el DBA y su personal tienen acceso al gestor de datos almacenados para la creación y actualización de la estructura real de las tablas.

#### Problemas Resueltos

**1.1.** ¿Qué tipo de usuario usualmente realizaría las siguientes funciones para un sistema de inventario en una empresa grande?

a.  Crear un reporte mensual del valor actual del inventario.

b.  Actualizar el número en stock para artículos específicos recibidos en un envío.

c.  Cancelar la cuenta de usuario para un empleado que acaba de jubilarse.

d.  Cambiar la estructura de la base de datos de inventario para incluir más información sobre cada artículo.

e.  Responder una solicitud telefónica sobre el número de un artículo particular que actualmente está en stock.

\-

a.  Programador de aplicaciones.

b.  Usuario final.

c.  DBA o la persona en el equipo designada con ese trabajo.

d.  DBA o la persona en el equipo designada con ese trabajo.

e.  Usuario final.

**1.2.** Considere una base de datos de inventario en una fábrica de gabinetes de cocina que contiene información de partes (número de parte, descripción, color, tamaño, número en stock, etc.) e información de proveedores (nombre, dirección, orden de compra, etc.). Indique para cada usuario, un empleado de cuentas por pagar, un supervisor de línea y un empleado de recepción, a qué elementos debería poder acceder y cuáles debería poder modificar.

[p. 19]

| Usuario | Nivel de permiso |
|--------------------------|----------------------------------------------|
| a\. Empleado de cuentas por pagar | Debería poder acceder y modificar todos los datos. |
| b\. Supervisor de línea | Necesita acceder, pero no modificar, información de partes. Probablemente no necesita tener acceso a ninguna información de proveedores excepto quizás el nombre. |
| c\. Empleado de recepción | Necesita poder acceder y modificar información de partes, como el número en stock. Debería poder acceder pero no modificar información de proveedores. |

**1.3.** ¿Por qué los sistemas cliente/servidor se han vuelto tan prevalentes en el mundo empresarial?

Primero, el avance de la tecnología de hardware permite que incluso las organizaciones pequeñas compren servidores potentes a un costo razonable. Tanto el software fácil de usar como los recursos de red también están dentro de un rango de precio razonable. Estos sistemas proporcionan un alto nivel de rendimiento mientras permiten capacidades confiables de respaldo y seguridad.

**1.4.** ¿Cuáles son algunas diferencias en temas de seguridad entre sistemas de usuario único y multiusuario?

Los sistemas de usuario único a menudo se mantienen seguros simplemente cerrando con llave la puerta de la habitación que contiene la computadora. Los sistemas multiusuario, ya sean independientes o cliente/servidor, deben emplear algún tipo de protección por contraseña que permita diferentes niveles de acceso a diferentes usuarios. Mientras que los temas de respaldo y recuperación son similares, tener usuarios concurrentes resulta en la necesidad de protección de transacciones, manejo de interbloqueos (deadlock) y bloqueo. Estos temas se discuten en el Capítulo 6.

**1.5.** Una junta escolar a nivel de condado quiere crear una base de datos distribuida para información de estudiantes. Describa cómo podría diseñarse. ¿Cómo sería diferente si quisieran un sistema centralizado?

Para un sistema distribuido, cada escuela local guardaría información sobre todos sus estudiantes en un servidor ubicado dentro de la escuela. Todas las bases de datos de estudiantes serían accesibles desde la oficina central y desde las otras escuelas para que el personal en cualquier ubicación pudiera encontrar información sobre cualquier estudiante. Si la junta escolar quisiera un sistema centralizado, necesitarían tener un servidor, probablemente ubicado en la oficina central, y luego permitir acceso a esa base de datos desde cada escuela local.

**1.6.** Especifique si cada sistema probablemente sería de una, dos o tres capas.

a.  Una artista mujer diseña y vende joyería y accesorios por correo o en ferias artesanales. Trabaja desde su casa.

b.  La junta escolar del ejemplo anterior que tiene un sistema centralizado con cada escuela local accediendo datos desde el servidor en la oficina central.

Debido a que la mujer en (a) trabaja desde su casa, probablemente tiene una computadora con la información de clientes y financiera en un DBMS. Este sería un sistema de una sola capa.

[p. 20]

La junta escolar en (b) tiene un DBMS. Cada escuela estaría ejecutando el software cliente del DBMS desde el servidor central. Este es un sistema de dos capas.

**1.7.** Una artista mujer diseña y vende joyería y accesorios por correo o en ferias artesanales. Trabaja desde su casa. Actualmente tiene una lista de correos en un archivo de procesamiento de palabras que usa para enviar catálogos. También guarda nombres y direcciones de proveedores y clientes en archivos de procesamiento de palabras y prepara facturas usando una hoja de cálculo, y guarda sus ingresos y gastos en otra hoja de cálculo. ¿Cuáles serían las ventajas para ella de diseñar y usar un DBMS para todos sus registros?

Primero, se le ayudaría con el mantenimiento adecuado de sus datos. Actualmente corre el riesgo de perder dinero porque podría no registrar cada transacción con cada cliente en su hoja de cálculo contable. Es posible que la dirección para la misma persona en la lista de catálogos y la lista de clientes pueda ser diferente. También podría no ser capaz de demostrar que ha cobrado todo el impuesto sobre ventas requerido por el gobierno. Si usara un DBMS, reduciría la redundancia y estaría asegurada de la precisión de sus registros. Segundo, tendría un acceso más rápido a sus datos. Podría preparar su declaración de impuestos más fácilmente con todas las consultas respondidas rápidamente. Usar el DBMS mejoraría la completitud de sus datos. Tercero, podría mantener la seguridad de sus datos. El DBMS puede limitar el acceso mediante el uso de contraseñas y también proporcionar respaldo y recuperación fáciles de sus registros.

**1.8.** Dado este modelo para una base de datos universitaria, ¿es jerárquico, de red o relacional?

[Descripción del diagrama: Un diagrama que muestra relaciones entre entidades. NÚMERO_DE_CURSO y TÍTULO_DE_CURSO están vinculados. NÚMERO_DE_CURSO apunta a DATOS_DE_FACULTAD mediante "impartiendo muchos cursos". TÍTULO_DE_CURSO apunta a ESTUDIANTE 1 mediante "teniendo muchos estudiantes". TÍTULO_DE_CURSO apunta a DÍA Y HORA (y NÚMERO_DE_SALÓN) mediante "teniendo muchos días y horarios y salones". DATOS_DE_FACULTAD apunta a ESTUDIANTE 1 mediante "teniendo muchos instructores". ESTUDIANTE 1 apunta a ESTUDIANTE N mediante "teniendo muchos estudiantes".]

[Figura p.1-8]
<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_1-8.png" width="80%">
  <div style="font-size: 0.9em; margin-top: 0.5em;"></div>
</div>

Este es un modelo de red porque cada nodo puede apuntar a varios otros y puede ser apuntado por varios otros.

[p. 21] **1.9.** ¿Cómo se organizarían los datos en la pregunta anterior si fueran relacionales?

| FACULTY NAME | COURSE1 | COURSE2 | COURSE3 |
|--------------|---------|---------|---------|
|              |         |         |         |
|              |         |         |         |

| COURSE NAME | COURSE NUMBER | DAY | TIME | ROOM NUMBER | MAX ENROLLMENT |
|-------------|---------------|-----|------|-------------|----------------|
|             |               |     |      |             |                |
|             |               |     |      |             |                |

| STUDENT NAME | COURSE1 | COURSE2 | COURSE3 |
|--------------|---------|---------|---------|
|              |         |         |         |
|              |         |         |         |

Se organizaría en una variedad de tablas como se muestra arriba. Los elementos aún estarían conectados por el número de curso.

**1.10.** Diseñe el esquema para un sistema que guarda información para una liga de fútbol juvenil.

Se muestra un esquema de muestra a continuación. Todas las tablas estarían conectadas por números de team_ID. Usando el modelo relacional, sería fácil acceder a los nombres de jugadores y entrenadores en cualquier equipo dado.

| **PLAYER INFORMATION** | **TEAM INFORMATION** | **COACH INFORMATION** |
|------------------------|----------------------|-----------------------|
| PLAYER_ID              | TEAM_ID              | COACH_ID              |
| NAME                   | TEAM_NAME            | NAME                  |
| ADDRESS                | TEAM_SPONSOR         | ADDRESS               |
| CITY                   | PRACTICE_LOCATION    | CITY                  |
| STATE                  | WINS                 | STATE                 |
| ZIP                    | LOSSES               | ZIP                   |
| PHONE                  |                      | PHONE                 |
| TEAM_ID                |                      | TEAM_ID               |

**1.11.** Explique la diferencia entre el DDL y el DML.

DDL o lenguaje de definición de datos, se utiliza para manejar la estructura de la base de datos, para crear o eliminar las tablas junto con las columnas y tipos de datos de las columnas dentro de las tablas. [p. 22] El DDL usualmente es utilizado solo por el DBA y sus asistentes. DML o lenguaje de manipulación de datos, consiste en declaraciones para acceder, recuperar o actualizar datos dentro de tablas existentes y predefinidas. El DML se utiliza para formular consultas interactivas o consultas desde dentro de programas de aplicación.

**1.12.** ¿Usaría el usuario el DML o el DDL para realizar cada tarea? (a) Actualizar el promedio de calificaciones de un estudiante, (b) definir una nueva tabla de cursos, (c) agregar una columna a la tabla de estudiantes.

a.  Actualizar el GPA de un estudiante se realizaría mediante el uso del DML. Esta actividad es parte de la manipulación de datos dentro de tablas actualmente definidas.

b.  y **c.** Definir una nueva tabla y agregar una columna a una tabla existente implicaría el uso del DDL. Crear tablas y establecer los atributos son parte de la definición de datos.

**1.13.** Dé ejemplos de la vista interna, de la comunidad y externa de la base de datos del equipo descrita en 1.10.

El nivel interno sería la ubicación física de la información en el medio de almacenamiento y el número de bytes utilizados por cada elemento.

La vista de la comunidad sería una vista conceptual de las tablas y la información contenida en ellas. Por ejemplo,

| **COACH_ID** | **NAME**         | **TEAM_ID** |
|--------------|------------------|-------------|
| 21           | Phil Johnson     | 13          |

La vista externa dependería del usuario. Los encargados de registros entenderían que podrían calcular las posiciones examinando los registros de victorias/derrotas de cada equipo. Los entrenadores sabrían que podrían acceder a los números de teléfono de todos los jugadores en su equipo.

**1.14.** Dé ejemplos de la vista interna, de la comunidad y externa de la base de datos universitaria descrita en 1.9.

El nivel interno sería la ubicación física de la información en el medio de almacenamiento y el número de bytes utilizados por cada elemento.

La vista de la comunidad sería una vista conceptual de las tablas y la información contenida en ellas. Por ejemplo,

| **COURSE_NAME**       | **COURSE_NUMBER** | **DAY**  | **TIME** |
|-----------------------|-------------------|----------|----------|
| Intro a Base de Datos | CS474             | Mar/Jue  | 13:00    |

La vista externa dependería del usuario. Los miembros de la facultad entenderían que podrían acceder a listas de estudiantes en los cursos. La oficina de registro sabría que debería publicar la lista de clases cerradas.

**1.15.** Explique la diferencia entre independencia de datos lógica y física.

La independencia de datos física está relacionada con el almacenamiento real de los datos en el medio de almacenamiento. La forma en que los datos se almacenan en bits y bytes no debería afectar el acceso de los usuarios a esos datos. La independencia de datos lógica usualmente se relaciona con las diferentes vistas lógicas de los datos disponibles para diferentes usuarios. Si un usuario ve nombre y dirección del cliente sin toda la información de facturación, eso no significa que la información de facturación no se esté almacenando. Otro usuario debería poder ver e incluso cambiar esta información sin corromper la base de datos en su conjunto.

[p. 23]

**1.16.** ¿Qué módulos de software utiliza el DBMS para proteger la precisión de la base de datos?

El DBMS utiliza el diccionario de datos, el gestor en tiempo de ejecución y el gestor de datos almacenados. Se accede al diccionario de datos para verificar que las solicitudes sean legales para esa base de datos particular. Por ejemplo, si el usuario quiere todas las partes que son rojas, el atributo color debe existir realmente dentro de la tabla. El gestor en tiempo de ejecución luego procesa las consultas reales, recuperando la información solicitada o actualizando la tabla. El gestor de datos almacenados, accesado por el DBA, puede usar funciones del sistema operativo o puede manejar tareas por su cuenta. Este módulo también mantiene actualizado el diccionario de datos siguiendo las declaraciones DDL.

#### Problemas Suplementarios

**1.17.** ¿Qué tipo de usuario realizaría las siguientes funciones para un sistema de facturación en una empresa grande?

a.  Responder una llamada de un cliente sobre el saldo actual debido en su cuenta.

b.  Escribir un programa para generar facturas mensuales.

c.  Desarrollar un esquema para un nuevo tipo de sistema de facturación.

**1.18.** Considere una base de datos en una universidad que contiene información sobre estudiantes (nombre, número de ID, horario de cursos, calificaciones, etc.), clases (número, nombre, lista de clase) y facultad (nombre, número de ID, horario de cursos, salario, etc.). Indique para cada usuario a qué elementos debería poder acceder y cuáles debería poder modificar.

a.  Miembro de la facultad

b.  Empleado en la oficina de registro

c.  Estudiante

d.  Empleado de nómina

**1.19.** Considere la universidad discutida en el problema anterior. ¿Su base de datos probablemente sería centralizada o distribuida? ¿Qué factores desconocidos podrían hacer una diferencia en el diseño?

**1.20.** Especifique si cada sistema sería de una, dos o tres capas. (a) Un sindicato de mecánicos permite a cada grupo local guardar sus propios registros en su propio tipo de DBMS. La sede central del sindicato quiere permitir acceso desde cualquier local a cualquier local. (b) Una práctica médica con 3 doctores y 2 ubicaciones guarda registros centralizados accesados por terminales en cada ubicación.

**1.21.** Una pequeña escuela privada quiere llevar un registro de estudiantes, cursos y facultad. También tiene un número de padres y contribuyentes privados. Describa las ventajas para esta escuela de usar un DBMS para todos sus registros.

[p. 24]

**1.22.** Dado este modelo para una base de datos de partes, ¿es jerárquico, de red o relacional?

[Descripción del diagrama: Una estructura de árbol jerárquico. La caja superior está etiquetada como "NOMBRE_DE_PARTE". Flechas apuntan hacia abajo desde "NOMBRE_DE_PARTE" a tres cajas: "COLOR", "COSTO" y "NOMBRE_DEL_PROVEEDOR". Flechas apuntan hacia abajo desde "NOMBRE_DEL_PROVEEDOR" a dos cajas: "DIRECCIÓN_DEL_PROVEEDOR" y "TELÉFONO_DEL_PROVEEDOR".]

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/mata-toledo_2000_figure_p-1.22.png" width="80%">
  <div style="font-size: 0.9em; margin-top: 0.5em;"></div>
</div>


**1.23.** Describa cómo se vería si fuera relacional. ¿Por qué el modelo relacional sería mejor para esta aplicación?

**1.24.** Diseñe el esquema para guardar información sobre flores para una empresa de pedidos por correo, incluyendo información sobre cada tipo de flor, el método de entrega (semillas, bulbo, maceta, etc.) y las zonas de temperatura en las que cada flor crecerá.

**1.25.** ¿Usaría el usuario el DML o el DDL para realizar cada tarea? (a) Definir una nueva tabla para guardar necesidades de suelo y luz, (b) agregar una flor al inventario, (c) agregar una nueva zona a la tabla de zonas.

**1.26.** Dé ejemplos de la vista interna, de la comunidad y externa de la base de datos de flores descrita en 1.21.

**1.27.** ¿Por qué es importante mantener la independencia de datos en una base de datos?

#### Respuestas a los Problemas Suplementarios

**1.17.** a. Usuario final. b. Programador de aplicaciones. c. DBA o designado en ese departamento.

**1.18.** **a.** El miembro de la facultad no debería poder cambiar nada. Debería tener acceso a información de clases, pero no a información de estudiantes. Debería tener acceso solo a sus propios datos de facultad.

[p. 25]

**b.** El empleado en la oficina de registro debería poder acceder y cambiar toda la información de clases. Debería poder acceder a información de estudiantes y cambiar solo los datos relacionados con calificaciones. Debería poder acceder y cambiar información de clases de facultad, pero no datos de salario de facultad.

**c.** El estudiante solo debería poder acceder a información de clases respecto a día y hora y si está abierta o cerrada. No debería tener acceso a ninguna lista de clases ni a ninguna información de facultad. Tampoco debería tener permiso para ver datos de otros estudiantes excepto los suyos propios.

**d.** El empleado de nómina probablemente no necesite acceso a datos de estudiantes. Debería poder acceder y cambiar datos de facultad. En algunos casos, podría necesitar acceder pero no cambiar información de clases.

**1.19.** A primera vista, uno pensaría que la base de datos universitaria sería centralizada con toda la información de cursos, estudiantes y facultad guardada en un solo lugar. Sin embargo, varios factores podrían cambiar esta decisión. Primero, si hay varios campus sucursales, cada campus podría querer guardar sus propios datos de estudiantes. Segundo, por razones de seguridad, diferentes oficinas en diferentes ubicaciones podrían querer guardar datos. Por ejemplo, la oficina de recursos humanos guardaría toda la información de facultad, la oficina de registro guardaría la información de cursos y calificaciones de estudiantes, y otra información de estudiantes podría ser guardada por asuntos estudiantiles. Con este tipo de distribución, el DBMS necesitaría poder acceder información en cualquier ubicación con la autorización adecuada.

**1.20.** **a.** Tres capas debido al middleware requerido para conectar diferentes tipos de DBMS. **b.** Dos capas.

**1.21.** Primero, la escuela estaría asegurada del mantenimiento adecuado de los datos. Todas las tablas con información personal serían precisas y estarían actualizadas. El DBMS haría cumplir la consistencia de información sobre todos los estudiantes, padres, facultad y contribuyentes. Segundo, el DBMS proporcionaría acceso rápido a los datos. Elementos como transcripciones de estudiantes, listas de cursos y números de teléfono de padres serían accesibles instantáneamente. Los registros financieros estarían disponibles para escrutinio por parte de los contribuyentes. Tercero, se mantiene la seguridad de los datos. Con un DBMS, los funcionarios de la escuela estarían seguros de que toda la información permanecería confidencial y segura.

**1.22.** Es jerárquico porque el acceso a cada elemento es solo a través de un camino comenzando desde arriba. No se podría acceder al costo sin conocer el nombre de la parte.

**1.23.** Estaría en tablas como se muestra a continuación. Las tablas estarían conectadas por el número de ID del proveedor. El modelo relacional sería mejor porque cualquier elemento puede ser accesado rápidamente de una variedad de maneras. Por ejemplo, se podría listar el costo de todos los artículos de un proveedor particular.

[p. 26]

| PART_NAME | COLOR | COST | SUPPLIER_ID |
|-----------|-------|------|-------------|
|           |       |      |             |

| SUPPLIER_ID | SUPPLIER_ADDRESS | SUPPLIER_PHONE |
|-------------|------------------|----------------|
|             |                  |                |

**1.24.** El esquema podría verse como la tabla a continuación. Cada flor está conectada al tipo de zona en la que crece y a la forma en que se entrega por los IDs respectivos.

| DELIVERY INFORMATION | FLOWER INFORMATION | ZONE INFORMATION |
|---------------------|-------------------|------------------|
| DELIVERY_ID | ID | ZONE_ID |
| CATEGORY | COMMON_NAME | LOWEST_TEMP |
| SIZE | LATIN_NAME | HIGHEST_TEMP |
| | HIGHEST_TEMP_ZONE_ID | |
| | LOWEST_TEMP_ZONE_ID | |
| | DELIVERY_ID | |
| | LIGHT_NEEDS | |
| | SOIL_TYPE | |

**1.25.** **a.** Definir una nueva tabla implicaría el uso del DDL. Crear la tabla y establecer los atributos son parte de la definición de datos. **b.** y **c.** Ingresar información de nueva zona y flor se realizaría mediante el uso del DML. Ambas actividades implican manipular datos dentro de tablas actualmente establecidas.

**1.26.** El nivel interno sería la ubicación física de la información en el medio de almacenamiento y el número de bytes utilizados por cada elemento.

La vista de la comunidad sería una vista conceptual de las tablas y la información contenida en ellas. Por ejemplo,

| DELIVERY_ID | CATEGORY | SIZE         |
|-------------|----------|--------------|
| 1           | maceta   | 1.5 pulgadas |

La vista externa dependería del usuario. El empleado de envíos entendería que la Hiedra Thorndale es tipo de entrega 1, que debería enviarse en una maceta. El personal de help desk respondería a una pregunta de que las plantas de Hiedra Thorndale pueden plantarse a pleno sol, sombra parcial o sombra.

[p. 27]

**1.27.** La independencia de datos es importante por tres razones. Primero, a nivel físico, el DBA debería poder cambiar la estructura interna de una base de datos sin alterar la vista de la comunidad o las vistas externas. Segundo, a nivel lógico, el código de aplicación o las consultas de usuario no deberían necesitar ser alteradas debido a cambios en la representación o almacenamiento de los datos. Tercero, la gestión de los datos es más fácil cuando el software del DBMS está separado de los datos reales. Diferentes usuarios pueden acceder a los datos concurrentemente y sentirse confiados de que el DBMS está manteniendo la integridad y precisión de los datos.

------------------------------------------------------------------------

**Nota sobre derechos de autor:** Esta traducción se proporciona exclusivamente con fines educativos y de investigación académica. El contenido original está protegido por derechos de autor © Mata-Toledo, R., & Cushman, P. (2000). *Schaum's outline of fundamentals of relational databases*. McGraw-Hill.

---

[^1]: E. F. Codd, *The Relational Model of Data for Large Shared Data Banks*. This model is further explained in E. F. Codd, *The Relational Model for Database Management Version 2*, Addison-Wesley, Reading, MA, 1990. <> 
[^2]: Dionysios C. Tsichritzis y Anthonly Klug (eds.), *The ANSI/X3/SPARC DBMS Framework: Report of the Study Group on Data Base Management Systems, Information Systems* 3, 1978. <https://sigmodrecord.org/publications/sigmodRecord/8207/pdfs/984555.1108830.pdf> 

