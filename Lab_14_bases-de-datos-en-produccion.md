### Laboratorio 14: Bases de Datos en Producción

**Dr. Jesús Zavala Ruiz**  
**Ultima actualización:** 4 de julio de 2026  

---

#### 1. Introducción

Hasta ahora, usted ha trabajado en un entorno académico controlado. Ha diseñado esquemas, escrito consultas SQL y optimizado bases de datos. Sin embargo, en esta sesión final, el objetivo cambia de paradigma: dejaremos el entorno de desarrollo para adentrarnos en la realidad de un entorno de producción. 

Una base de datos en producción no solo debe ser funcional; debe ser segura, auditable, resiliente y operacionalmente sostenible. Los datos que alberga son, en la mayoría de los casos, el activo más valioso de la organización. Este laboratorio le exigirá integrar todos los conocimientos adquiridos y aplicarlos bajo un enfoque de defensa en profundidad (*defense in depth*), donde la seguridad no es un producto ni una capa única, sino un proceso continuo que abarca desde el silicio del servidor hasta la última consulta SQL.

Imagine esta escena: son las 9:00 AM del lunes. Suena su teléfono. Es Sofía Vargas, Project Manager de Innovatech Solutions, una empresa mexicana especializada en soluciones de gestión empresarial para el sector retail. La voz de Sofía es urgente pero calmada: 

> *"Necesitamos que el nuevo sistema de fidelización de clientes esté en producción en tres semanas. El equipo de desarrollo terminó la aplicación y las pruebas funcionaron bien en QA, pero el equipo de seguridad detectó que nuestra infraestructura base no cumple con los estándares de la industria. No podemos permitirnos un incidente; los datos personales y financieros de nuestros clientes son nuestro activo más valioso."*

Usted ha sido contratado como Consultor Senior en Seguridad de Bases de Datos. Su misión no es solo hacer que PostgreSQL funcione, sino asegurarla integralmente.

Para enfrentar este desafío, se integrará a un equipo multidisciplinario. Es fundamental comprender que en un entorno real, la base de datos no existe en el vacío; es el núcleo de un ecosistema complejo:

-   **Roberto Hernández (DBA Senior):** 15 años de experiencia. Será su mentor técnico en las entrañas de PostgreSQL.  
-   **Ana Martínez (DBA Junior):** Recién egresada, entusiasta, conoce la teoría pero necesita guía en producción.  
-   **Carlos Ramírez (Developer Lead):** Lidera el equipo de desarrollo. Necesita que la base de datos sea rápida y confiable.  
-   **Laura Sánchez (Developer):** Desarrolladora backend. Trabajarán juntos en la integración de la aplicación.  
-   **Miguel Torres (QA/Tester):** Obsesionado con encontrar bugs. Probará cada configuración de seguridad para intentar romperla.  
-   **Patricia Flores (SysAdmin):** Experta en Linux. Configurará el sistema operativo, la red y el almacenamiento.  
-   **Javier López (Security Officer):** Auditará cada decisión técnica para asegurar el cumplimiento normativo.  
-   **Sofía Vargas (Project Manager):** Coordina los tiempos, entregables y la comunicación con la directiva.  

**Nota Técnica: El peligro del "Organizational Lock-in"**

> En la administración de sistemas y bases de datos en producción, existe un riesgo tan crítico como cualquier vulnerabilidad técnica: el **organizational lock-in** (también conocido como *vendor lock-in interno* o dependencia de un solo punto de falla humano). Este fenómeno ocurre cuando el conocimiento especializado sobre la seguridad, configuración y operación de un sistema crítico reside exclusivamente en una sola persona. En la industria de la ciberseguridad, esto se cuantifica mediante el **"Bus Factor"** (Factor Autobús): el número mínimo de personas que, si fueran atropelladas por un autobús (o simplemente renunciaran, se enfermaran o fueran comprometidas), dejarían al proyecto sin capacidad operativa. Un bus factor de 1 es una vulnerabilidad organizacional grave.

Los riesgos de asignar la seguridad de la base de datos a una sola persona incluyen:

1. **Pérdida catastrófica de conocimiento:** Si el DBA o administrador único abandona la organización, se lleva consigo las contraseñas maestras, las ubicaciones de las claves de cifrado, los procedimientos de recuperación ante desastres y la lógica detrás de las políticas de SELinux y `pg_hba.conf`. La organización queda expuesta y paralizada.  
2. **Insider Threat amplificado:** Una sola persona con acceso total y sin supervisión representa el vector de ataque más peligroso. No existe *segregación de funciones* (*segregation of duties*), lo que facilita el fraude, el sabotaje o la exfiltración de datos sin detección.  
3. **Ausencia de revisión por pares:** Sin un segundo par de ojos, los errores de configuración (como un `pg_hba.conf` permisivo o una clave LUKS mal gestionada) permanecen sin detectar hasta que un atacante los explota.  
4. **Bloqueo operativo:** La organización queda secuestrada por la disponibilidad de esa persona. Vacaciones, enfermedades o emergencias personales se convierten en incidentes de continuidad del negocio.  
5. **Incumplimiento normativo:** Estándares como PCI-DSS, ISO 27001, HIPAA y NIST 800-53 exigen explícitamente la segregación de funciones, la rotación de conocimiento y la documentación de procedimientos críticos. Un bus factor de 1 viola estos controles.  

**Mitigación:** Por esta razón, este laboratorio presenta un **equipo multidisciplinario** (DBA Senior, DBA Junior, SysAdmin, Security Officer, etc.). La seguridad en producción no es un acto heroico individual, sino un proceso colectivo, documentado y auditable. Las mejores prácticas dictan que:  
- Todo conocimiento crítico debe estar documentado en manuales operativos (*runbooks*) accesibles.  
- Las credenciales y claves deben gestionarse mediante bóvedas de secretos (como HashiCorp Vault o FreeIPA).  
- Al menos dos personas deben ser capaces de ejecutar cualquier procedimiento crítico (principio de doble control o *four-eyes*).  
- La rotación de roles y el entrenamiento cuzado (*cross-training*) son obligatorios, no opcionales.  

Recuerde: **un sistema seguro operado por una sola persona no es seguro; es una bomba de tiempo organizacional.**

##### 1.3. Separación de Entornos

En el contexto de la ciberseguridad y la administración de sistemas, el término ***hardening*** (endurecimiento o aseguramiento) se refiere al proceso sistemático de reforzar la seguridad de un sistema operativo, aplicación o base de datos. 

Por defecto, los sistemas se instalan y configuran priorizando la **usabilidad y la compatibilidad**, lo que a menudo implica dejar puertos abiertos, servicios activos y cuentas predeterminadas. El *hardening* busca revertir esto: su objetivo principal es **reducir la superficie de ataque** (*attack surface*). Esto se logra deshabilitando servicios innecesarios, aplicando parches de seguridad, eliminando cuentas por defecto y forzando políticas de acceso estrictas.

Si la instalación de un servidor es como construir una casa, el *hardening* es el proceso de instalar cerraduras de alta seguridad, alarmas y rejas, asegurándose de que ninguna ventana quede abierta por descuido.

En la industria, este proceso no se hace a ciegas; se guía por estándares y perfiles de seguridad internacionales reconocidos, como los **CIS Benchmarks** (Center for Internet Security), las guías de **NIST** o normativas de cumplimiento como **PCI-DSS** e **ISO 27001**. A lo largo de este laboratorio, aplicaremos *hardening* en múltiples capas (Defensa en Profundidad), desde el núcleo del sistema operativo (Rocky Linux) hasta el motor de base de datos (PostgreSQL).

Carlos Ramírez (Developer Lead) le pregunta en la primera reunión: *"¿Podemos probar los cambios de seguridad directamente en el servidor de producción para ir más rápido?"*. Su respuesta debe ser un rotundo NO.

En la práctica profesional, es obligatorio separar las bases de datos en al menos tres entornos:

-   **Desarrollo (DEV):** Donde los ingenieros construyen nuevas funcionalidades. Usa datos anonimizados o sintéticos. La seguridad es relajada y las caídas son aceptables.  
-   **Pruebas / QA (TEST):** Donde se validan las funcionalidades antes de producción. Usa copias recientes de producción. La seguridad es media y se permiten ventanas de mantenimiento.  
-   **Producción (PROD):** El sistema real que atiende a los usuarios finales. Contiene datos vivos y críticos. La seguridad es máxima (todas las medidas de este laboratorio se aplican aquí). La disponibilidad debe ser del 99.9% o superior.  

*Regla fundamental:* Los cambios nunca pasan directamente de desarrollo a producción. Siempre deben atravesar el entorno de pruebas.

Es importante señalar que este laboratorio se enfoca exclusivamente en el endurecimiento y aseguramiento de la base de datos en el entorno de producción. Otros aspectos del ciclo de vida del sistema corresponden a cursos subsecuentes del plan de estudios de la Licenciatura:

-   **Modelado de datos y diseño de esquemas:** Bases de Datos (curso actual).  
-   **Diseño de la arquitectura de la aplicación y flujos de datos:** Análisis y Diseño de Sistemas.  
-   **Implementación de pipelines CI/CD, contenedores y separación automática de entornos:** Ingeniería de Software.  

Este laboratorio es una pieza fundamental del rompecabezas, pero la excelencia en ingeniería de sistemas requiere la integración coordinada de todas estas disciplinas.

#### 2. El Stack Empresarial Mínimo: Soberanía Tecnológica

Patricia Flores (SysAdmin) y Javier López (Security Officer) han definido la arquitectura. En el contexto actual, la dependencia de software propietario y el *vendor lock-in* representan riesgos financieros y estratégicos. Para Innovatech Solutions, se ha establecido el siguiente triplete tecnológico como el stack empresarial mínimo de referencia para garantizar la soberanía tecnológica:

##### 2.1. Rocky Linux 10 como Base

**Rocky Linux** es una distribución de Linux de grado empresarial, construida como reemplazo binario 100% compatible con Red Hat Enterprise Linux (RHEL). Provee la estabilidad, compatibilidad y soporte a largo plazo de RHEL, manteniendo la soberanía tecnológica y eliminando costos de licenciamiento, mientras ofrece seguridad nativa a nivel de kernel mediante SELinux.

##### 2.2. FreeIPA como Directorio de Identidades

**FreeIPA** (*Free Identity, Policy, Audit*) es una solución integrada de gestión de identidades de código abierto. Combina LDAP, Kerberos, DNS y gestión de certificados. En una organización con cientos de empleados, gestionar cuentas locales en cada servidor es insostenible y peligroso. FreeIPA centraliza la autenticación, permitiendo la revocación inmediata de accesos y el cumplimiento de políticas corporativas.

##### 2.3. PostgreSQL como Motor de Base de Datos

**PostgreSQL** es el Sistema Gestor de Bases de Datos relacional de código abierto más avanzado. Garantiza integridad ACID, extensibilidad y mecanismos de seguridad avanzados. Para este laboratorio, utilizaremos la base de datos de demostración **`pagila`** (un port a PostgreSQL de la famosa Sakila), que simula una tienda de renta de DVDs con datos sensibles de clientes, inventario y pagos.

#### 3. Panorama de Amenazas

Antes de configurar cualquier parámetro, Javier López (Security Officer) le pide una reunión. Quiere entender contra qué están defendiendo los datos. Los vectores de ataque más comunes en entornos productivos incluyen:

-   **Inyección SQL (SQLi):** Explotación de vulnerabilidades en la capa de aplicación para manipular consultas SQL.  
-   **Fuerza Bruta y Credential Stuffing:** Intentos masivos de autenticación contra el servicio expuesto.  
-   **Escalamiento de Privilegios:** Abuso de configuraciones laxas de roles para obtener acceso administrativo.  
-   **Sniffing de Red (MitM):** Interceptación de consultas y datos sensibles transmitidos en texto plano.  
-   **Denegación de Servicio (DoS):** Saturación de conexiones o ejecución de consultas maliciosas.  
-   **Explotación del Sistema Operativo:** Compromiso del host subyacente para acceder directamente a los archivos de datos (`$PGDATA`).  
-   **Robo de Backups:** Acceso no autorizado a copias de seguridad no cifradas.  
-   **Insider Threat:** Empleados con acceso legítimo que abusan de sus privilegios.  

#### 4. Fase 1: Creación del Entorno y Endurecimiento del Sistema Operativo

Roberto Hernández (DBA Senior) le advierte: *"La seguridad de una base de datos en producción comienza por el sistema operativo. Si un atacante compromete el SO, todas nuestras medidas a nivel de PostgreSQL son inútiles."*

##### 4.1. Requisitos y Creación de la Máquina Virtual

Para este laboratorio, usted provisionará su propio servidor. Dado que ejecutaremos PostgreSQL, FreeIPA (como cliente) y servicios de cifrado, necesitaremos una Máquina Virtual (MV) con las siguientes características:

*   **CPU:** 2 a 4 núcleos.
*   **RAM:** 6 GB a 8 GB (FreeIPA y PostgreSQL consumen memoria).
*   **Red:** Configurar la interfaz de red con la IP estática **`192.168.122.25`** (máscara `255.255.255.0`).
*   **Disco 1 (Sistema):** 40 GB para el sistema operativo Rocky Linux 10.
*   **Disco 2 (Datos):** 20 GB adicionales (sin formatear) para crear la partición cifrada con LUKS y montar ahí el directorio `$PGDATA`.

*Nota didáctica:* En la vida real, el servidor Tang (para NBDE) y el servidor FreeIPA estarían en máquinas separadas. Para este "monolito de laboratorio", simularemos estos servicios en la misma MV. Dado que no contamos con un servidor DNS externo, configure el archivo `/etc/hosts` para resolver los nombres ficticios:

```bash
echo "192.168.122.25 pgsql.ejemplo.com tang.ejemplo.com ipa.ejemplo.com" | sudo tee -a /etc/hosts
```

> **Nota Técnica: Uso de dominios en documentación vs. producción**
> 
> En este laboratorio utilizamos el dominio `ejemplo.com` (y sus subdominios como `ipa.ejemplo.com` o `pgsql.ejemplo.com`). De acuerdo con los estándares de la IETF (RFC 2606 y RFC 6761), estos dominios están reservados exclusivamente para fines de documentación, pruebas y entornos académicos, garantizando que no existan colisiones con dominios reales en internet ni se exponga tráfico accidentalmente.
> 
> **Sin embargo, es fundamental aclarar que en un entorno de producción real**, como el que requiere *Innovatech Solutions*, la organización debe utilizar su **dominio legal y corporativo real** (por ejemplo, `innovatech.com.mx`). El uso del dominio real es obligatorio en producción para garantizar la resolución DNS interna, la emisión de certificados SSL/TLS válidos por autoridades certificadoras (CA) públicas o privadas, y el cumplimiento estricto de las políticas de seguridad, trazabilidad y auditoría de la empresa.

> **Nota Técnica: El peligro del "Organizational Lock-in" y el Factor Autobús**
> 
> En la administración de sistemas y bases de datos en producción, existe un riesgo tan crítico como cualquier vulnerabilidad técnica: el **organizational lock-in** (también conocido como *vendor lock-in interno* o dependencia de un solo punto de falla humano). Este fenómeno ocurre cuando el conocimiento especializado sobre la seguridad, configuración y operación de un sistema crítico reside exclusivamente en una sola persona.
> 
> En la industria de la ciberseguridad, esto se cuantifica mediante el **"Bus Factor"** (Factor Autobús): el número mínimo de personas que, si fueran atropelladas por un autobús (o simplemente renunciaran, se enfermaran o fueran comprometidas), dejarían al proyecto sin capacidad operativa. Un bus factor de 1 es una vulnerabilidad organizacional grave.
> 
> **Los riesgos de asignar la seguridad de la base de datos a una sola persona incluyen:**
> 
> 1. **Pérdida catastrófica de conocimiento:** Si el DBA o administrador único abandona la organización, se lleva consigo las contraseñas maestras, las ubicaciones de las claves de cifrado, los procedimientos de recuperación ante desastres y la lógica detrás de las políticas de SELinux y `pg_hba.conf`. La organización queda expuesta y paralizada.
> 2. **Insider Threat amplificado:** Una sola persona con acceso total y sin supervisión representa el vector de ataque más peligroso. No existe *segregación de funciones* (*segregation of duties*), lo que facilita el fraude, el sabotaje o la exfiltración de datos sin detección.
> 3. **Ausencia de revisión por pares:** Sin un segundo par de ojos, los errores de configuración (como un `pg_hba.conf` permisivo o una clave LUKS mal gestionada) permanecen sin detectar hasta que un atacante los explota.
> 4. **Bloqueo operativo:** La organización queda secuestrada por la disponibilidad de esa persona. Vacaciones, enfermedades o emergencias personales se convierten en incidentes de continuidad del negocio.
> 5. **Incumplimiento normativo:** Estándares como PCI-DSS, ISO 27001, HIPAA y NIST 800-53 exigen explícitamente la segregación de funciones, la rotación de conocimiento y la documentación de procedimientos críticos. Un bus factor de 1 viola estos controles.
> 
> **Mitigación:** Por esta razón, este laboratorio presenta un **equipo multidisciplinario** (DBA Senior, DBA Junior, SysAdmin, Security Officer, etc.). La seguridad en producción no es un acto heroico individual, sino un proceso colectivo, documentado y auditable. Las mejores prácticas dictan que:
> - Todo conocimiento crítico debe estar documentado en *runbooks* accesibles.
> - Las credenciales y claves deben gestionarse mediante bóvedas de secretos (como HashiCorp Vault o FreeIPA).
> - Al menos dos personas deben ser capaces de ejecutar cualquier procedimiento crítico (principio de *four-eyes* o doble control).
> - La rotación de roles y el *cross-training* son obligatorios, no opcionales.
> 
> Recuerde: **un sistema seguro operado por una sola persona no es seguro; es una bomba de tiempo organizacional.**

##### 4.2. Instalación de Rocky Linux 10 y Modo FIPS

Descargue la ISO de Rocky Linux 10 y cree la MV. Durante la instalación, Patricia Flores (SysAdmin) le indica que, para cumplir con normativas gubernamentales, el sistema debe operar en modo FIPS (Federal Information Processing Standards).

*Regla de oro:* En RHEL/Rocky Linux 10, FIPS **solo puede habilitarse durante la instalación** agregando `fips=1` en los parámetros del kernel (presionando `e` en el menú de arranque y editando la línea que inicia con `linux`). No se puede habilitar posteriormente sin reinstalar.

Una vez instalado, inicie sesión como el usuario `alumno` (contraseña: `uamIztapalapa`) y actualice el sistema:

```bash
sudo dnf update -y
sudo reboot
```
##### 4.3. Políticas Criptográficas y Cumplimiento (OpenSCAP)

Rocky Linux 10 centraliza la seguridad criptográfica a nivel de sistema operativo. Verifique que el sistema esté usando políticas robustas:

```bash
## Ver política actual
update-crypto-policies --show

## Si no está en FIPS o FUTURE, establézcalo (requiere reboot)
sudo update-crypto-policies --set FUTURE
sudo reboot
```

Javier López exige un escaneo de cumplimiento normativo utilizando el estándar CIS (Center for Internet Security). Instale y ejecute OpenSCAP:

```bash
sudo dnf install -y openscap-scanner scap-security-guide

## Escanear el sistema contra el perfil CIS Server Level 1
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
--results scan-results.xml --report scan-report.html \
/usr/share/xml/scap/ssg/content/ssg-rl10-ds.xml
```

*Nota didáctica:* El archivo `scan-report.html` es un reporte visual que puede presentar a la directiva de Innovatech para demostrar qué reglas de seguridad fallan y cuáles pasan.

##### 4.4. Control de Aplicaciones (fapolicyd) e Integridad (AIDE)

¿Qué pasa si un atacante logra subir un script malicioso o un binario compilado al servidor? Para evitarlo, implementaremos `fapolicyd`, un framework de *allowlisting* que bloquea la ejecución de cualquier binario que no esté en la base de datos oficial de paquetes RPM:

```bash
sudo dnf install -y fapolicyd
sudo systemctl enable --now fapolicyd
```

Además, para detectar si un atacante modifica binarios del sistema o archivos de configuración, utilizaremos AIDE (Advanced Intrusion Detection Environment):

```bash
sudo dnf install -y aide
## Inicializar la base de datos de integridad (tarda varios minutos)
sudo aide --init
sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

## Programar verificación diaria
sudo systemctl enable --now aidecheck.timer
```

##### 4.5. SELinux, Firewall y Endurecimiento de SSH

SELinux debe permanecer en modo *Enforcing*. Verifique su estado con `sestatus`. Si PostgreSQL se instala en un directorio no estándar (ej. `/datos/pgsql`), debe contextualizar el directorio:

```bash
sudo dnf install -y policycoreutils-python-utils
sudo semanage fcontext -a -t postgresql_db_t "/datos/pgsql(/.*)?"
sudo restorecon -Rv /datos/pgsql
```

Configure el firewall para restringir el puerto 5432 únicamente a la red local de aplicaciones (`192.168.122.0/24`):

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.122.0/24" port port="5432" protocol="tcp" accept'
sudo firewall-cmd --reload
```

Finalmente, endurezca SSH editando `/etc/ssh/sshd_config`: deshabilite el login root por contraseña, habilite solo autenticación por llaves públicas y limite los intentos de autenticación.

#### 5. Fase 2: Instalación de PostgreSQL y Carga de Datos

##### 5.1. Instalación del Motor de Base de Datos

Instale PostgreSQL 15 o 16 desde los repositorios oficiales o PGDG:

```bash
sudo dnf install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql
```

##### 5.2. Carga de la Base de Datos

Descargue los scripts de `pagila`, cree la base de datos y cárguelos en el motor:

```bash
cd /tmp
sudo dnf install -y unzip wget
wget https://github.com/devrimgunduz/pagila/archive/refs/heads/master.zip
unzip master.zip
cd pagila-master

## Crear la base de datos (paso crítico que no debe omitirse)
sudo -u postgres psql -c "CREATE DATABASE pagila;"

## Cargar esquema y datos
sudo -u postgres psql -d pagila -f pagila-schema.sql
sudo -u postgres psql -d pagila -f pagila-data.sql
```

Verifique que las tablas (film, customer, payment, etc.) se hayan creado correctamente.

#### 6. Fase 3: Control de Acceso de Red y Autenticación Local

##### 6.1. Configuración de postgresql.conf

Edite el archivo de configuración principal (`/var/lib/pgsql/data/postgresql.conf`):

```ini
## Limitar exposición de red a la IP asignada al servidor
listen_addresses = '192.168.122.25'

## Migrar a SCRAM-SHA-256 (estándar actual, obligatorio para FIPS)
password_encryption = scram-sha-256

## Límite de conexiones
max_connections = 100

## Logging
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d.log'
```

##### 6.2. Restricción en pg_hba.conf

Edite `/var/lib/pgsql/data/pg_hba.conf`. Elimine las reglas permisivas por defecto y configure autenticación estricta:

```text
## TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     peer
host    pagila          all             192.168.122.0/24        scram-sha-256
host    all             all             0.0.0.0/0               reject
```

Reinicie PostgreSQL: `sudo systemctl restart postgresql`.

#### 7. Fase 4: Gestión de Identidades y Privilegios (Modelo de Roles)

Conéctese como superusuario: `sudo -u postgres psql`.

##### 7.1. Revocación de Permisos en el Esquema Público

Esta es una acción crítica en versiones modernas de PostgreSQL para mitigar ataques de "esquema público inseguro":

```sql
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON SCHEMA public FROM PUBLIC;
```

##### 7.2. Creación de Roles Granulares

```sql
-- Rol de solo lectura
CREATE ROLE app_readonly NOLOGIN;
GRANT CONNECT ON DATABASE pagila TO app_readonly;
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

-- Rol de lectura/escritura
CREATE ROLE app_readwrite NOLOGIN;
GRANT CONNECT ON DATABASE pagila TO app_readwrite;
GRANT USAGE ON SCHEMA public TO app_readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_readwrite;

-- Usuario de aplicación local (para pruebas antes de LDAP)
CREATE ROLE alumno LOGIN PASSWORD 'uamIztapalapa' IN ROLE app_readwrite;
```

#### 8. Fase 5: Cifrado en Tránsito (SSL/TLS)

##### 8.1. Generación de Certificados

```bash
sudo -u postgres openssl req -new -x509 -days 365 -nodes \
    -out /var/lib/pgsql/data/server.crt \
    -keyout /var/lib/pgsql/data/server.key \
    -subj "/CN=pgsql.ejemplo.com"

sudo chmod 600 /var/lib/pgsql/data/server.key
sudo chown postgres:postgres /var/lib/pgsql/data/server.crt /var/lib/pgsql/data/server.key
```

##### 8.2. Habilitación y Verificación

En `postgresql.conf`:

```ini
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_min_protocol_version = 'TLSv1.2'
```

Reinicie PostgreSQL y verifique la conexión cifrada desde un cliente:

```bash
psql "host=192.168.122.25 dbname=pagila user=alumno sslmode=verify-full sslrootcert=/var/lib/pgsql/data/server.crt"
```

Dentro de `psql`, ejecute:

```sql
SELECT s.ssl, s.version, s.cipher 
FROM pg_stat_ssl s JOIN pg_stat_activity a ON s.pid = a.pid 
WHERE a.usename = current_user;
```

#### 9. Fase 6: Mecanismos de Cifrado Nativo de PostgreSQL

##### 9.1. Cifrado de Contraseñas

Verifique que todas las contraseñas estén almacenadas con SCRAM-SHA-256:

```sql
SELECT usename, passwd LIKE 'SCRAM-SHA-256$%' AS es_scram 
FROM pg_shadow WHERE passwd IS NOT NULL;
```

##### 9.2. Extensión pgcrypto (Cifrado a Nivel de Columna)

*Nota didáctica:* A diferencia del cifrado en reposo (LUKS) que protege todo el disco, `pgcrypto` permite cifrar datos específicos (ej. números de tarjetas) antes de almacenarlos, de modo que ni siquiera un DBA con acceso root a la base de datos pueda leerlos sin la clave maestra.

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE public.datos_sensibles_pagila (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customer(customer_id),
    dato_confidencial BYTEA,
    fecha_creacion TIMESTAMP DEFAULT NOW()
);

-- Inserción de datos cifrados
INSERT INTO public.datos_sensibles_pagila (customer_id, dato_confidencial)
VALUES (
    1,
    pgp_sym_encrypt('Tarjeta de crédito: 4111-1111-1111-1111', 'clave_maestra_innovatech')
);

-- Consulta de datos cifrados
SELECT customer_id, pgp_sym_decrypt(dato_confidencial, 'clave_maestra_innovatech') AS dato_descifrado 
FROM public.datos_sensibles_pagila;
```

##### 9.3. Cifrado de Backups y Consideraciones sobre TDE

PostgreSQL Community **no incluye TDE (Transparent Data Encryption) nativo**. Las alternativas son LUKS (que veremos en la Fase 8) o extensiones de terceros. Para backups, utilice `pg_basebackup` combinado con cifrado externo:

```bash
## Crear directorio de backups con permisos adecuados
sudo mkdir -p /var/lib/pgsql/backups
sudo chown postgres:postgres /var/lib/pgsql/backups

## Realizar backup cifrado
sudo -u postgres pg_basebackup -D - -Ft -z | \
openssl enc -aes-256-cbc -salt -pbkdf2 -out /var/lib/pgsql/backups/pagila_backup.tar.gz.enc \
-pass pass:clave_backup_muy_segura
```

#### 10. Fase 7: Autenticación Centralizada con FreeIPA / LDAP

##### 10.1. Configuración del Cliente LDAP

Instale las utilidades de cliente de FreeIPA y enrolle el servidor al dominio `ejemplo.com` (asumiendo que el servidor IPA está accesible en la red o se simula el enrolamiento):

```bash
sudo dnf install -y freeipa-client
## Nota: En un entorno real, esto requeriría un servidor IPA activo. 
## Para el laboratorio, se asume que la directiva LDAP se configura manualmente o contra un servidor externo.
sudo ipa-client-install --domain=ejemplo.com --realm=EJEMPLO.COM --server=ipa.ejemplo.com
```

##### 10.2. Integración con pg_hba.conf

Edite `pg_hba.conf` para autenticar contra FreeIPA usando LDAPS (puerto 636):

```text
host    pagila    all    192.168.122.0/24    ldap \
    ldapserver=ipa.ejemplo.com \
    ldapport=636 \
    ldapscheme=ldaps \
    ldapprefix="uid=" \
    ldapsuffix=",cn=users,cn=accounts,dc=ejemplo,dc=com"
```

##### 10.3. Prueba de Revocación Centralizada

1. Cree un usuario en FreeIPA: `ipa user-add jdoe --first=John --last=Doe`
2. Agréguelo al grupo de bases de datos: `ipa group-add-member pg_readonly --users=jdoe`
3. Conecte desde un cliente: `psql -h 192.168.122.25 -U jdoe -d pagila`
4. **Prueba de fuego:** Deshabilite el usuario en FreeIPA: `ipa user-disable jdoe`
5. Intente reconectar inmediatamente. **Debe fallar**, demostrando la revocación centralizada instantánea.

#### 11. Fase 8: Cifrado en Reposo con NBDE (Clevis y Tang)

Si un atacante obtiene acceso físico al servidor o roba los discos, el cifrado a nivel de DBMS no es suficiente. Sin embargo, usar un `keyfile` local para desbloquear LUKS es una mala práctica. Implementaremos **NBDE (Network-Bound Disk Encryption)**.

##### 11.1. Despliegue del Servidor Tang

En el mismo servidor (monolito), despliegue el servicio Tang:

```bash
sudo dnf install -y tang
sudo systemctl enable --now tangd.socket
sudo firewall-cmd --add-service=tang --permanent && sudo firewall-cmd --reload
```

##### 11.2. Vinculación del Volumen LUKS con Clevis

Suponiendo que ya tiene un disco adicional (`/dev/sdb`) formateado con LUKS para alojar `$PGDATA`:

```bash
sudo dnf install -y clevis clevis-luks clevis-dracut

## Vincular la partición LUKS al servidor Tang local (usando localhost para evitar problemas de DNS en el arranque temprano)
sudo clevis luks bind -d /dev/sdb tang '{"url":"http://localhost"}'

## Regenerar el initramfs para que el sistema pueda desbloquear el disco en el arranque
sudo dracut -f --regenerate-all
```

*Nota didáctica:* Al reiniciar, la partición `/datos/pgsql` se montará automáticamente siempre que el servidor Tang responda localmente. Si se roba el disco físico, los datos son ilegibles.

#### 12. Fase 9: Auditoría y Monitoreo Continuo

##### 12.1. Logs Nativos

En `postgresql.conf`, configure la auditoría básica:

```ini
log_connections = on
log_disconnections = on
log_statement = 'ddl'
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
```

##### 12.2. Extensión pgAudit

Para auditoría detallada a nivel de objeto (requerida por normativas como PCI-DSS):

```bash
sudo dnf install -y pgaudit_15
```

En `postgresql.conf`:

```ini
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl, role, misc'
pgaudit.log_parameter = on
```

Reinicie PostgreSQL y verifique los logs:

```bash
sudo tail -f /var/lib/pgsql/data/log/postgresql-*.log | grep AUDIT
```

Excelente adición. En un entorno de producción, la seguridad y el rendimiento son dos caras de la misma moneda. Un sistema seguro pero inoperante por lentitud es un fracaso, al igual que un sistema rápido pero vulnerable. 

#### 13. Fase 10: Tuning y Optimización de Rendimiento en Producción

##### 13.1. Contexto: El Desafío de los 200 Usuarios Concurrentes

Durante la reunión de planificación, Carlos Ramírez (Developer Lead) expresa una preocupación crítica: *"El sistema de fidelización atenderá a unos 200 usuarios concurrentes en horas pico, principalmente ejecutando reportes de ventas y consultas complejas sobre la base `pagila`. Si la base de datos se congela, perdemos clientes."*

Roberto Hernández (DBA Senior) asiente y toma la palabra: *"La configuración por defecto de PostgreSQL está diseñada para ser conservadora y funcionar en cualquier hardware, desde una Raspberry Pi hasta un servidor de 128 núcleos. Para producción, debemos ajustar los parámetros de memoria, WAL (Write-Ahead Logging) y el planificador de consultas para aprovechar nuestra Máquina Virtual de 8 GB de RAM."*

##### 13.2. Ajustes de Memoria en `postgresql.conf`

La memoria es el recurso más crítico para el rendimiento de PostgreSQL. Roberto guía al equipo en la modificación de los siguientes parámetros en `/var/lib/pgsql/data/postgresql.conf`:

###### 13.2.1. Caché Compartida y Caché Efectiva

*   **`shared_buffers`**: Es la memoria que PostgreSQL dedica exclusivamente a cachear datos de disco. La regla general es asignar el 25% de la RAM total del sistema. Para nuestra MV de 8 GB, asignaremos 2 GB.
    ```ini
    shared_buffers = 2GB
    ```
*   **`effective_cache_size`**: Este parámetro no asigna memoria, sino que le indica al planificador de consultas cuánta memoria está disponible en el sistema (incluyendo el caché del sistema operativo). Esto ayuda a PostgreSQL a decidir si usar índices o hacer barridos de tabla. Se recomienda entre el 50% y 75% de la RAM total.
    ```ini
    effective_cache_size = 6GB
    ```

###### 13.2.2. Memoria de Trabajo y Mantenimiento

*   **`work_mem`**: Es la memoria utilizada por cada operación de ordenamiento (ORDER BY) o hash (JOIN). Si se establece muy alto y hay 200 conexiones concurrentes, el sistema podría quedarse sin RAM (OOM Killer). Un valor conservador pero eficiente es adecuado.
    ```ini
    work_mem = 32MB
    ```
*   **`maintenance_work_mem`**: Memoria destinada a tareas de mantenimiento como `VACUUM`, `CREATE INDEX` o `ALTER TABLE`. Al no haber cientos de estas operaciones simultáneas, se puede asignar un valor mayor para acelerar el mantenimiento.
    ```ini
    maintenance_work_mem = 512MB
    ```

##### 13.3. Optimización de WAL y Checkpoints

El Write-Ahead Logging (WAL) garantiza la durabilidad (la 'D' de ACID), pero una configuración por defecto puede generar cuellos de botella en escrituras intensivas.

*   **`max_wal_size`**: Define el tamaño máximo de los archivos WAL antes de forzar un checkpoint. Aumentarlo reduce la frecuencia de los checkpoints, mejorando el rendimiento de escritura.
    ```ini
    max_wal_size = 4GB
    ```
*   **`checkpoint_completion_target`**: Indica cuánto tiempo debe tardar un checkpoint en completarse como fracción del intervalo entre checkpoints. Un valor de 0.9 "estira" las escrituras de los checkpoints en el tiempo, evitando picos de I/O que saturen el disco.
    ```ini
    checkpoint_completion_target = 0.9
    ```
*   **`wal_buffers`**: Memoria para los datos de WAL que aún no se han escrito en disco.
    ```ini
    wal_buffers = 64MB
    ```

##### 13.4. Ajuste del Planificador de Consultas (Query Planner)

PostgreSQL asume por defecto que el almacenamiento es un disco duro mecánico (HDD) lento. Dado que Innovatech Solutions utiliza almacenamiento moderno (SSD o SAN de alto rendimiento), debemos ajustar las "constantes" del planificador para que prefiera los índices sobre los barridos secuenciales.

*   **`random_page_cost`**: Reduce el costo relativo de las lecturas aleatorias (índices) frente a las secuenciales.
    ```ini
    random_page_cost = 1.1
    ```
*   **`effective_io_concurrency`**: Indica al sistema cuántas solicitudes de I/O simultáneas puede emitir. Para discos SSD, este valor debe ser alto.
    ```ini
    effective_io_concurrency = 200
    ```

##### 13.5. Diagnóstico Continuo con `pg_stat_statements`

*"Lo que no se mide, no se puede mejorar"*, advierte Javier López (Security Officer). Para auditar el rendimiento, Roberto activa la extensión `pg_stat_statements`, que rastrea estadísticas de ejecución de todas las consultas SQL.

Dado que ya estamos cargando `pgaudit` en la Fase 9, debemos agregar esta extensión a la misma directiva:

```ini
## En postgresql.conf
shared_preload_libraries = 'pgaudit, pg_stat_statements'
```

Reinicie PostgreSQL y active la extensión en la base de datos `pagila`:
```bash
sudo systemctl restart postgresql
sudo -u postgres psql -d pagila -c "CREATE EXTENSION IF NOT EXISTS pg_stat_statements;"
```

Para identificar las consultas más lentas o costosas, el equipo de desarrollo ejecutará esta consulta periódicamente:
```sql
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

##### 13.6. Consideraciones a Nivel de Sistema Operativo (HugePages)

Roberto hace una última recomendación para el SysAdmin, Patricia Flores: *"Para evitar que el kernel de Rocky Linux fragmente la memoria y para reducir la sobrecarga de la TLB (Translation Lookaside Buffer) al manejar nuestros 2 GB de `shared_buffers`, deberíamos configurar **HugePages** en el sistema operativo."*

Aunque la configuración profunda de HugePages excede el alcance de este laboratorio, Patricia debe asegurar que en el servidor de producción real se configuren en `/etc/sysctl.conf`:
```ini
vm.nr_hugepages = 1024  ## Ajustar según shared_buffers
```
Y configurar PostgreSQL para usarlas:
```ini
huge_pages = try
```

#### 14. Entregables y Rúbrica

##### 14.1. Informe Técnico de Hardening y Tuning (Actualizado)
Usted deberá entregar un informe que contenga:
1.  **Justificación estratégica:** Stack empresarial, soberanía tecnológica y defensa en profundidad.
2.  **Mapa de amenazas y mitigaciones:** Cómo cada fase (SO, Red, DBMS) mitiga los vectores de ataque.
3.  **Evidencias de Seguridad:** Capturas de `sestatus`, `clevis luks list`, `pgAudit`, y autenticación LDAP.
4.  **Evidencias de Tuning y Rendimiento:** 
    *   Captura del archivo `postgresql.conf` con los parámetros de memoria (`shared_buffers`, `work_mem`, etc.).
    *   Salida de la consulta a `pg_stat_statements` demostrando el rastreo de consultas.

#### 13. Entregables

##### 13.1. Informe Técnico de Hardening

Usted deberá entregar un informe ejecutivo y técnico que contenga:
1.  **Justificación estratégica:** Por qué se eligió el stack Rocky Linux + FreeIPA + PostgreSQL.
2.  **Mapa de amenazas:** Cómo cada fase mitiga los vectores de ataque identificados en la Sección 3.
3.  **Evidencias técnicas:** Capturas de pantalla de `sestatus`, `update-crypto-policies --show`, `clevis luks list`, `fapolicyd` activo, `aide` inicializado, y logs de `pgAudit`.
4.  **Prueba de Cifrado:** Script SQL demostrando la inserción y consulta de datos cifrados con `pgcrypto` sobre la base de datos `pagila`.
5.  **Prueba de LDAP:** Captura mostrando la autenticación exitosa vía FreeIPA y el rechazo tras la deshabilitación del usuario.

##### 13.2. Script de Validación Automatizada

Entregue un script en Bash (`validar_hardening.sh`) que audite automáticamente el cumplimiento:

```bash
#!/bin/bash
echo "=== Validación de Hardening PostgreSQL ==="
echo -n "SELinux Enforcing: "; getenforce | grep -q "Enforcing" && echo "OK" || echo "FALLA"
echo -n "SCRAM-SHA-256: "; sudo -u postgres psql -t -c "SHOW password_encryption;" | grep -q "scram-sha-256" && echo "OK" || echo "FALLA"
echo -n "SSL habilitado: "; sudo -u postgres psql -t -c "SHOW ssl;" | grep -q "on" && echo "OK" || echo "FALLA"
echo -n "LUKS/NBDE activo: "; cryptsetup status pgdata_crypt &>/dev/null && echo "OK" || echo "FALLA"
echo -n "pgAudit cargado: "; sudo -u postgres psql -t -c "SHOW shared_preload_libraries;" | grep -q "pgaudit" && echo "OK" || echo "FALLA"
echo "=== Validación Completa ==="
```

#### 14. Rúbrica de Autoevaluación

| Criterio | Peso |
|---|---|
| Seguridad a nivel de SO (FIPS, NBDE, fapolicyd, AIDE, SELinux) | 15% |
| Control de red, `pg_hba.conf`, SCRAM y SSL/TLS | 15% |
| Mecanismos de cifrado nativo (`pgcrypto`) y gestión de roles | 15% |
| Integración y configuración de autenticación centralizada con FreeIPA/LDAP | 15% |
| Auditoría (`pgAudit`) y logging | 10% |
| **Tuning de Rendimiento (`postgresql.conf`, `pg_stat_statements`)** | **15%** |
| Claridad técnica, justificación estratégica y formato del informe | 15% |


#### 15. Conclusiones del Laboratorio

La ejecución integral de este laboratorio de cierre valida la transición desde los fundamentos académicos hacia las responsabilidades operativas de un Administrador de Bases de Datos (DBA) e Ingeniero de Sistemas en entornos productivos. Los conocimientos adquiridos a lo largo del curso encuentran su aplicación más crítica en la **protección, optimización y operación continua de datos empresariales**.

Como resultado de la implementación del stack tecnológico en el escenario de Innovatech Solutions, se establecen las siguientes conclusiones técnicas y estratégicas:

1.  **La seguridad es un proceso continuo, no una configuración estática.** El endurecimiento inicial del sistema (FIPS, SELinux, `fapolicyd`) es insuficiente por sí solo. La implementación de herramientas de auditoría y verificación de integridad como `pgAudit`, AIDE y OpenSCAP demuestra que la postura de seguridad requiere monitoreo constante y validación periódica contra benchmarks (CIS/PCI-DSS).
2.  **La defensa en profundidad es arquitectónicamente obligatoria.** La efectividad de la seguridad no reside en una sola capa, sino en la integración del stack completo: el control de acceso a nivel de kernel (SELinux), la gestión centralizada de identidades (FreeIPA/LDAP), el cifrado en tránsito (SSL/TLS) y el cifrado a nivel de aplicación (`pgcrypto`). Cada capa mitiga vectores de ataque específicos que las demás no pueden cubrir.
3.  **El rendimiento y la seguridad son objetivos complementarios.** Las prácticas de tuning de PostgreSQL (configuración de `shared_buffers`, `work_mem`, `effective_io_concurrency`) y la habilitación de extensiones de diagnóstico como `pg_stat_statements` demuestran que es posible optimizar el rendimiento de consultas sin comprometer la postura de seguridad, siempre que se apliquen bajo políticas de auditoría estrictas.
4.  **La soberanía tecnológica es una ventaja estratégica y operativa.** La implementación exitosa del triplete Rocky Linux 10 + FreeIPA + PostgreSQL confirma la viabilidad de construir infraestructura de clase empresarial, altamente segura y auditable, eliminando la dependencia de *vendor lock-in* y reduciendo costos de licenciamiento sin sacrificar cumplimiento normativo (FIPS 140-3).
5.  **La disponibilidad y confidencialidad de los datos definen la responsabilidad del DBA.** La implementación de cifrado en reposo mediante NBDE (Clevis/Tang) y LUKS, combinada con estrategias de backups cifrados y rotación de claves, establece que la protección del activo más valioso de la organización requiere una gestión proactiva del ciclo de vida de los datos, desde su almacenamiento hasta su destrucción segura.
6.  **La alineación con los requerimientos del cliente y las normativas de cumplimiento (compliance) es el motor de la arquitectura.** En un entorno real, las decisiones técnicas no se toman en el vacío; responden directamente a las exigencias de auditoría, protección de datos personales y niveles de servicio (SLAs) establecidos por la directiva del cliente. La ingeniería de sistemas exitosa es aquella que traduce estos requerimientos de negocio y legales en controles técnicos tangibles, verificables y auditables.

Este laboratorio consolida la competencia técnica necesaria para diseñar, desplegar y mantener sistemas de bases de datos resilientes, asegurando que la infraestructura de datos no solo sea funcional, sino inherentemente segura, optimizada y preparada para satisfacer las demandas críticas de un entorno de producción real.

#### 16. Referencias

Center for Internet Security. (2023). *CIS PostgreSQL 16 benchmark v1.0.0*. https://www.cisecurity.org/benchmark/postgresql

Kumar, V., & Mehra, G. (2024). *RedHat Enterprise Linux 9 for beginners: A comprehensive guide for learning, administration, and deployment*. BPB Publications.

National Institute of Standards and Technology. (2019). *Security requirements for cryptographic modules* (FIPS Publication 140-3). U.S. Department of Commerce. https://doi.org/10.6028/NIST.FIPS.140-3

PostgreSQL Global Development Group. (2024). *PostgreSQL 16 documentation*. https://www.postgresql.org/docs/current/

Red Hat, Inc. (2026). *Red Hat Enterprise Linux 10 configuring and managing networking*. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/10/html-single/configuring_and_managing_networking/

Red Hat, Inc. (2026). *Red Hat Enterprise Linux 10 security hardening*. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/10/html-single/security_hardening/

Rocky Enterprise Software Foundation. (2026). *Rocky Linux 10 release notes*. https://docs.rockylinux.org/release_notes/
