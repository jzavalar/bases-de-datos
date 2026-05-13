### Track Intensivo de 3 Días 
### Preparación de Entorno para Explorar DBMS con la Base de Datos Pagila

### Laboratorios 01 a 04

### Licenciatura en Computación - UAM Iztapalapa

**UEA:** Bases de Datos  
**Profesor:** dr. Jesús Zavala Ruiz  
**Última actualización:** 12 de Mayo de 2026

**Objetivo:**

Tener listo el sistema local y acceso a sandbox online para comenzar a explorar qué es un DBMS con la base de datos Pagila

------------------------------------------------------------------------

#### Visión General del Track Intensivo

Este track comprimido está diseñado para que estudiantes sin experiencia previa en Linux o virtualización puedan preparar su entorno de desarrollo en **3 días intensivos**, con el objetivo de comenzar a explorar conceptos de sistemas gestores de bases de datos (DBMS) usando la base de datos de muestra Pagila.

| Día | Laboratorios | Objetivo del Día | Resultado Esperado |
|---------------|---------------|--------------------|----------------------|
| **Día 1** | Lab 01 + inicio Lab 02 | Instalar [Fedora](https://fedoraproject.org/) y preparar [virtualización](https://www.redhat.com/en/topics/virtualization/what-is-virtualization) | Fedora operativo + [KVM](https://linux-kvm.org/page/Main_Page)/[libvirt](https://libvirt.org/) configurado |
| **Día 2** | Lab 02 (completar) +<br>Lab 03 | Desplegar VM con PostgreSQL y herramientas | VM con [PostgreSQL](https://www.postgresql.org/) 16 + [DBeaver](https://dbeaver.io/)/[pgAdmin](https://www.pgadmin.org/) listos |
| **Día 3** | Lab 04 +<br>sandbox online | Cargar Pagila y<br>practicar [administración básica](https://neon.com/postgresql/administration) | Base de datos [Pagila](https://github.com/JamesRonsonOp/SQL_Training_with_Pagila) operativa localmente + acceso a sandbox online |

**Requisitos transversales**: 
- Computadora con ≥ 8 GB RAM,
- Espacio en dico ≥ 40 GB de almacenamiento libre,
- procesador con extensiones de virtualización (VT-x/AMD-V),
- conexión a internet estable,
- Memoria USB de ≥ 16 GB.

**Alternativa para hardware limitado**: Si tu equipo no cumple los requisitos, podrás usar exclusivamente cualquiera de los [sandboxes online](https://github.com/jzavalar/bases-de-datos/blob/main/02_recursos-de-aprendizaje_bases-de-datos.md) proporcionados. El track está diseñado para que ambos caminos (local y online) converjan en los mismos ejercicios de aprendizaje.

**Recomendaciones:** 
1. Mantenga una bitácora de lo que le da resultado y de lo que le da error para que pueda ir resolviendo los problemas. Se sugiere utilizar cualquier chatbot de IA, como [Qwen.ai](https://chat.qwen.ai/), como tutor, para resolver sus dudas de manera inmediata, conforme progrese.  
2. Conforme progrese vaya elaborando su lista de términos estilo vocabulario, para que haga su propio acordeón.  

------------------------------------------------------------------------

### Laboratorio 01: Instalación de Fedora Workstation 
### (Día 1 - Mañana)

#### Ficha Didáctica

| Elemento | Descripción |
|-------------------------------|----------------------------------------|
| Objetivo | Instalar Fedora Linux en el equipo personal, eligiendo entre dual boot con Windows o reemplazo total. |
| Público | Estudiantes sin experiencia previa en Linux. |
| Duración | 2.5 horas |
| Alcance | Instalación efectiva mediante Anaconda: particionamiento, usuario, gestor de arranque. |

#### Conceptos que Aprenderás

| Término | Definición Sencilla |
|------------------------|------------------------------------------------|
| UEFI | Firmware moderno que controla el arranque; reemplaza al BIOS antiguo. |
| ESP | Partición pequeña (100-500 MB) con cargadores de arranque; crítica en dual boot. |
| GRUB | Menú al encender que permite elegir entre sistemas operativos instalados. |
| LUKS | Sistema de cifrado de discos en Linux; protege datos ante robo físico. |
| Passphrase | Frase de contraseña para desbloquear disco cifrado; debe ser segura y memorable. |

#### Procedimiento Acelerado

#### Módulo 1: Preparación Rápida (30 min)

Desde Windows:

1. Verificar requisitos mínimos en la línea de comandos: 

```         
systeminfo | Select-String "System Type"  ### Debe decir: x64-based PC
Get-PSDrive C | Select-Object Free        ### Mínimo 20 GB libres
```

2. Crear USB de instalación

   - Descargar Fedora Media Writer: <https://github.com/FedoraQt/MediaWriter/releases/download/5.3.1/FedoraMediaWriter-win64-5.3.1.exe>  
   - Ejecutar y seguir asistente para crear USB booteable (Memoria USB ≥ 16 GB)  

3. Respaldar archivos importantes

   - Copiar documentos críticos a disco externo o nube

#### Módulo 2: Instalación Guiada (90 min)

1.  Reiniciar con USB insertado → Boot Menu (F12/F9 según fabricante)  
2.  Seleccionar opción "UEFI: [nombre del USB]"  
3.  En menú Fedora: "Start Fedora-Workstation-Live"  
4.  Ejecutar "Install to Hard Drive"  
    Configuración en Anaconda: 
    - Idioma: Español (México)  
    - Teclado: Latinoamericano  
    - Usuario: [tu_nombre] con contraseña segura + marcar "administrador"  
    Destino de instalación: 
    - Dual Boot: seleccionar disco → "Personalizado"  
      - ESP existente: /boot/efi, NO formatear  
      - Espacio no asignado: 
          - crear /boot (1 GB, ext4) y / (resto, ext4) 
    - Reemplazo Total: marcar "Cifrar mis datos" → establecer passphrase LUKS  
      - Confirmar cambios → "Comenzar instalación" (15-20 min)  

5.  Reiniciar, retirar USB, seleccionar "Fedora Linux" en GRUB  
6.  Iniciar sesión con tu usuario  

##### Módulo 3: Validación Inmediata (30 min)

Abrir terminal y ejecutar:

Verificar instalación

```         
cat /etc/fedora-release  ### Esperado: Fedora release 40 (Workstation Edition)
```

Actualizar sistema (rápido)

```         
sudo dnf update -y --refresh
```

Verificar conectividad

```         
ping -c 2 getfedora.org  ### Debe responder
```

Preparar para virtualización

```         
lscpu | grep Virtualization  ### Confirmar VT-x o AMD-V
```

[Solo dual boot] Verificar acceso a Windows

```         
lsblk -o NAME,FSTYPE | grep ntfs  ### Debe listar partición de Windows
```

#### Entregable Día 1 (Mañana)

Bitácora mínima (`lab01.md`) con: escenario elegido, evidencia de `cat /etc/fedora-release`, checklist de validación completado.

------------------------------------------------------------------------

### Laboratorio 02: Virtualización y PostgreSQL en VM 
### (Día 1 - Tarde + Día 2 - Mañana)

#### Ficha Didáctica

| Elemento | Descripción |
|-------------------------------|----------------------------------------|
| Objetivo | Configurar virtualización en Fedora y desplegar VM con Rocky Linux 10.1 + PostgreSQL 16. |
| Público | Estudiantes que completaron Lab 01. |
| Duración | 4 horas (Día 1 tarde: 2h |
| Alcance | KVM/libvirt, creación de VM, instalación de PostgreSQL y herramientas gráficas. |

#### Conceptos que Aprenderás

| Término | Definición Sencilla |
|------------------------|------------------------------------------------|
| KVM | Módulo del kernel que permite ejecutar máquinas virtuales con soporte de hardware. |
| libvirt | Herramienta que gestiona VMs de forma unificada y sencilla. |
| PGDG | Repositorio oficial de PostgreSQL con versiones actualizadas. |
| pg_hba.conf | Archivo que controla quién puede conectarse a PostgreSQL y cómo. |
| scram-sha-256 | Método seguro para verificar contraseñas; recomendado para producción. |

#### Procedimiento Acelerado

##### Módulo 1: Configurar Virtualización en Fedora (Día 1 - Tarde, 60 min)

En terminal de Fedora:

1. Instalar componentes de virtualización

   ```         
   sudo dnf groupinstall -y "Virtualization Host"
   sudo dnf install -y virt-manager virt-viewer
   ```

2. Habilitar servicio

   ```         
   sudo systemctl enable --now libvirtd
   ```

3. Configurar permisos de usuario

   ```         
   sudo usermod -aG kvm,libvirt $USER
   ```

   Cerrar sesión y volver a entrar para aplicar cambios

4. Verificar configuración

   ```         
   virsh list --all  ### Debe ejecutar sin error de permisos
   sudo virsh net-info default  ### Debe mostrar: Active: yes
   ```

##### Módulo 2: Crear VM con Rocky Linux 10.1 (Día 1 - Tarde, 60 min)

1. Descargar imagen ISO (si no se hizo previamente) y verificar integridad:

   ```         
   mkdir -p ~/isos && cd ~/isos
   wget -q https://download.rockylinux.org/pub/rocky/10.1/isos/x86_64/Rocky-10.1-x86_64-minimal.iso
   wget -q https://download.rockylinux.org/pub/rocky/10.1/isos/x86_64/Rocky-10.1-x86_64-minimal.iso.CHECKSUM
   sha256sum -c Rocky-10.1-x86_64-minimal.iso.CHECKSUM
   ```

2. Crear VM con virt-manager (interfaz gráfica):

   ```         
   virt-manager
   ```

   Nueva VM → ISO local → seleccionar Rocky-10.1-minimal.iso

   - Memoria: 2048 MB \| CPUs: 2 \| Disco: 20 GB qcow2

   - Nombre: rocky-postgres-lab \| Red: default (NAT)

     Iniciar instalación → Minimal Install → usuario "practicante" con sudo

   Alternativa línea de comandos:

   ```         
   sudo virt-install --name rocky-postgres-lab --memory 2048 --vcpus 2 \
     --disk size=20,format=qcow2 --cdrom ~/isos/Rocky-10.1-x86_64-minimal.iso \
     --network network=default --graphics none --noautoconsole
   ```

##### Módulo 3: Instalar PostgreSQL 16 en la VM (Día 2 - Mañana, 60 min)

1. Conectarse a la VM:

   ```         
   virsh domifaddr rocky-postgres-lab  ### Obtener IP
   ssh practicante@<IP_DE_LA_VM>
   ```

2. Dentro de la VM Rocky Linux:

   1. Actualizar e instalar PostgreSQL desde PGDG

      ```         
      sudo dnf update -y
      sudo rpm --import https://download.postgresql.org/pub/repos/yum/keys/PGDG-RPM-GPG-KEY-RHEL
      sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-10.1-x86_64/pgdg-redhat-repo-42.0-64.rhel10.1PGDG.noarch.rpm
      sudo dnf -qy module disable postgresql
      sudo dnf install -y postgresql16-server postgresql16-contrib postgresql16
      ```

   2. Inicializar y arrancar

      ```         
      sudo /usr/pgsql-16/bin/postgresql-16-setup initdb
      sudo systemctl enable --now postgresql-16
      sudo systemctl status postgresql-16  ### Verificar: active (running)
      ```

   3. Configurar acceso básico

      ```         
      sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'Admin2026!';"
      sudo nano /var/lib/pgsql/16/data/pg_hba.conf
      ```

      Cambiar líneas locales a: scram-sha-256

      ```         
      sudo systemctl reload postgresql-16
      ```

   4. Crear usuario y base para prácticas

      ```         
      sudo -u postgres psql << 'EOF'
      CREATE ROLE practicante_db WITH LOGIN PASSWORD 'Pract2026!' CREATEDB;
      CREATE DATABASE pagila_lab OWNER practicante_db;
      EOF
      ```

##### Módulo 4: Instalar Herramientas Gráficas (Día 2 - Mañana, 30 min)

Dentro de la VM:

1. DBeaver (cliente universal)

   ```         
   sudo dnf install -y https://dbeaver.io/files/dbeaver-ce-latest.x86_64.rpm
   ```

2. pgAdmin 4 (interfaz web oficial)

   ```         
   sudo dnf install -y https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-redhat-repo-2-1.noarch.rpm
   sudo dnf install -y pgadmin4
   sudo /usr/pgadmin4/bin/setup-web.sh  ### Seguir asistente
   ```

3. Probar conexión desde Fedora (host)

   ```         
   psql -h <IP_DE_LA_VM> -U practicante_db -d pagila_lab -c "SELECT version();"
   ```

#### Entregable Día 2 (Mañana)

Bitácora (`lab02_03.md`) con: parámetros de VM, comandos de instalación de PostgreSQL, evidencia de conexión exitosa (`SELECT version();`), archivo `conexion.txt` con parámetros de acceso.

------------------------------------------------------------------------

### Laboratorio 04: Exploración de DBMS con Pagila (Día 2 - Tarde + Día 3)

#### Ficha Didáctica

| Elemento | Descripción |
|-------------------------------|----------------------------------------|
| Objetivo | Cargar la base de datos Pagila y realizar ejercicios básicos de administración para explorar conceptos de DBMS. |
| Público | Estudiantes que completaron Labs 01-03. |
| Duración | 4.5 horas (Día 2 tarde: 2h |
| Alcance | Despliegue de Pagila, consultas básicas, respaldos simples, mantenimiento preventivo. |

#### Conceptos que Aprenderás

| Término | Definición Sencilla |
|------------------------|------------------------------------------------|
| Pagila | Base de datos de muestra que modela un sistema de renta de películas; incluye datos reales para practicar. |
| Esquema | Contenedor lógico dentro de una base de datos para organizar tablas; evita conflictos de nombres. |
| pg_dump | Herramienta para crear respaldos lógicos; exporta estructura y/o datos. |
| VACUUM | Proceso que libera espacio ocupado por datos eliminados; mantiene la base eficiente. |
| MVCC | Mecanismo de PostgreSQL que permite múltiples versiones de filas para concurrencia sin bloqueos. |

#### Procedimiento Acelerado

##### Módulo 1: Cargar Pagila (Día 2 - Tarde, 45 min)

1. Conectarse a la VM

   ```         
   ssh practicante@<IP_DE_LA_VM>
   cd ~
   ```

2. Descargar Pagila de alguno de los dos repositorios y cargar Pagila

   ```
   git clone --depth 1 https://github.com/devrimgunduz/pagila.git
   cd pagila
         
   git clone --depth 1 https://github.com/JamesRonsonOp/SQL_Training_with_Pagila.git
   cd SQL_Training_with_Pagila
   ```

3. Cargar esquema y luego datos

   ```         
   psql -U practicante_db -d pagila_lab -f pagila-schema.sql
   psql -U practicante_db -d pagila_lab -f pagila-data.sql
   ```

4. Verificar carga

   ```         
   psql -U practicante_db -d pagila_lab -c "\dt"
   ```

   Debe listar: actor, film, customer, rental, payment, etc.

5. Primera exploración: consultar datos reales

   ```         
   psql -U practicante_db -d pagila_lab << 'EOF'
   -- ¿Cuántas películas hay?
   SELECT COUNT(*) AS total_peliculas FROM film;
   
   -- ¿Cuáles son las 5 categorías más populares?
   SELECT c.name, COUNT(fc.film_id) AS total
   FROM category c
   JOIN film_category fc ON c.category_id = fc.category_id
   GROUP BY c.name ORDER BY total DESC LIMIT 5;
   
   -- ¿Qué película tiene más actores?
   SELECT f.title, COUNT(fa.actor_id) AS total_actores
   FROM film f
   JOIN film_actor fa ON f.film_id = fa.film_id
   GROUP BY f.film_id, f.title
   ORDER BY total_actores DESC LIMIT 3;
   EOF
   ```

##### Módulo 2: Ejercicios Básicos de Administración (Día 2 - Tarde, 45 min)

Conectarse a pagila_lab

   ```         
   psql -U practicante_db -d pagila_lab

   -- 1. Crear vista para reporte simple
   CREATE VIEW vista_peliculas_rentadas AS
   SELECT f.title, COUNT(r.rental_id) AS veces_rentada
   FROM film f
   JOIN inventory i ON f.film_id = i.film_id
   JOIN rental r ON i.inventory_id = r.inventory_id
   GROUP BY f.film_id, f.title;
   
   -- 2. Consultar la vista
   SELECT * FROM vista_peliculas_rentadas ORDER BY veces_rentada DESC LIMIT 10;

   -- 3. Crear respaldo lógico simple
   \q
   pg_dump -U practicante_db -h localhost -Fc pagila_lab > ~/pagila_backup.dump

   -- 4. Verificar respaldo
   pg_restore -l ~/pagila_backup.dump | head -20
   ```

##### Módulo 3: Mantenimiento y Diagnóstico Básico (Día 3 - Mañana, 45 min)

   ```         
   psql -U practicante_db -d pagila_lab
   
   -- 1. Verificar tamaño de tablas principales
   SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) AS tamano
   FROM pg_stat_user_tables
   WHERE schemaname = 'public'
   ORDER BY pg_total_relation_size(relid) DESC
   LIMIT 5;
   
   -- 2. Ejecutar mantenimiento en tablas críticas
   VACUUM ANALYZE film, rental, payment;
   
   -- 3. Verificar estadísticas post-mantenimiento
   SELECT relname, n_dead_tup, last_vacuum
   FROM pg_stat_user_tables
   WHERE relname IN ('film', 'rental', 'payment');
   
   -- 4. Explorar conexiones activas
   SELECT pid, usename, state, query_start
   FROM pg_stat_activity
   WHERE datname = 'pagila_lab' AND state != 'idle';
   ```

##### Módulo 4: Sandbox Online - Alternativa y Complemento (Día 3 - Mañana, 30 min)

Para estudiantes con hardware limitado o como complemento:

- Opción 1: PostgreSQL en la nube (gratuito)
   - <https://neon.tech/> → cuenta gratuita con PostgreSQL 16  
   - Crear proyecto → conectar con psql o DBeaver  
   - Cargar Pagila usando los scripts del repositorio GitHub

- Opción 2: Contenedor Docker en cualquier sistema (Ver guía [aquí](https://github.com/devrimgunduz/pagila))  
   - Instalar Docker Desktop (Windows/Mac/Linux)  
   - Ejecutar: 
   `docker run -d --name pagila-sandbox \     
   -e POSTGRES_PASSWORD=Pract2026 \     
   -p 5432:5432 postgres:16`\

      Conectar: `psql -h localhost -U postgres -d postgres`

- Opción 3: Entorno web interactivo 
   - <https://pgexercises.com/> → ejercicios con base de datos de ejemplo  
   - <https://www.db-fiddle.com/> → probar consultas SQL sin instalación

   Configuración común para sandbox: 
   - Host: [proporcionado por la plataforma]   
   - Puerto: 5432 (o el indicado)  
   - Usuario: [según plataforma]  
   - Base de datos: pagila_lab o equivalente

##### Módulo 5: Ejercicio Integrador Final (Día 3 - Tarde, 60 min)

Script de respaldo automatizado simple

```         
cat > ~/respaldo_pagila.sh << 'EOF'
#!/bin/bash
### Respaldo diario de pagila_lab
DB="pagila_lab"
USER="practicante_db"
DIR=~/backups
mkdir -p "$DIR"
pg_dump -U "$USER" -h localhost -Fc "$DB" > "$DIR/pagila_$(date +%F).dump"
find "$DIR" -name "pagila_*.dump" -mtime +3 -delete
echo "Respaldo completado: $(ls -1 $DIR/pagila_*.dump | wc -l) archivos retenidos"
EOF
chmod +x ~/respaldo_pagila.sh
~/respaldo_pagila.sh
```

Validación final: ejecutar checklist

```         
cat > ~/validar_entorno.sh << 'EOF'
#!/bin/bash
echo "=== Validación de entorno DBMS ==="
if psql -U practicante_db -d pagila_lab -c "SELECT 1;" &>/dev/null; then
  echo "[OK] PostgreSQL accesible"
else
  echo "[ERROR] Verificar conexión"
  exit 1
fi
if psql -U practicante_db -d pagila_lab -t -c "SELECT COUNT(*) FROM film;" | grep -q "[0-9]"; then
  echo "[OK] Pagila cargada correctamente"
else
  echo "[ERROR] Verificar carga de Pagila"
  exit 1
fi
if [ -f ~/pagila_backup.dump ]; then
  echo "[OK] Respaldo disponible"
else
  echo "[WARN] Crear respaldo con pg_dump"
fi
echo "=== Entorno listo para explorar DBMS ==="
EOF
chmod +x ~/validar_entorno.sh
~/validar_entorno.sh
```

#### Entregables Día 3

1.  **Bitácora final** (`lab04_pagila.md`) con:

    - Evidencia de carga exitosa de Pagila (`\dt` o consulta de conteo)
    - Resultados de 3 consultas exploratorias con interpretación breve
    - Estrategia de respaldos implementada (script `respaldo_pagila.sh`)
    - Reflexión: ¿qué aprendiste sobre qué es un DBMS después de estos ejercicios?

2.  **Validación ejecutable**: Salida de `~/validar_entorno.sh` mostrando al menos 2/3 [OK].

3.  **Acceso a sandbox online**: URL o configuración de la plataforma alternativa utilizada (si aplica).

------------------------------------------------------------------------

#### Calendario Detallado de 3 Días

```         
DÍA 1 - Instalación y Virtualización
┌─────────────────────────────────────────────────────┐
│ 09:00-11:30 | Lab 01: Fedora Installation           │
│ • Preparación y creación de USB (30 min)            │
│ • Instalación mediante Anaconda (90 min)            │
│ • Validación y actualización (30 min)               │
├─────────────────────────────────────────────────────┤
│ 14:00-16:00 | Lab 02 (parte 1): Virtualización      │
│ • Instalación de Gnome Boxes/KVM/libvirt (30 min)   │
│ • Configuración de permisos y red (30 min)          │
│ • Descarga de ISO Rocky Linux (inicio)              │
└─────────────────────────────────────────────────────┘

DÍA 2 - PostgreSQL y Herramientas
┌─────────────────────────────────────────────────────┐
│ 09:00-11:00 | Lab 02 (parte 2) + Lab 03 (inicio)    │
│ • Creación de VM con Rocky Linux (60 min)           │
│ • Instalación de PostgreSQL 16 (60 min)             │
├─────────────────────────────────────────────────────┤
│ 14:00-16:30 | Lab 04 (parte 1): Pagila              │
│ • Carga de esquema y datos (45 min)                 │
│ • Consultas exploratorias básicas (45 min)          │
│ • Respaldo lógico con pg_dump (30 min)              │
└─────────────────────────────────────────────────────┘

DÍA 3 - Exploración de DBMS y Cierre
┌─────────────────────────────────────────────────────┐
│ 09:00-10:30 | Lab 04 (parte 2): Mantenimiento       │
│ • Diagnóstico con vistas del sistema (30 min)       │
│ • VACUUM/ANALYZE y estadísticas (30 min)            │
│ • Sandbox online: configuración alternativa (30 min)│
├─────────────────────────────────────────────────────┤
│ 14:00-15:30 | Ejercicio integrador y cierre         │
│ • Script de respaldo automatizado (30 min)          │
│ • Validación final del entorno (30 min)             │
│ • Reflexión y entrega de bitácoras (30 min)         │
└─────────────────────────────────────────────────────┘
```

#### Soporte Rápido por Día

| Día | Problema Común | Solución Inmediata |
|----------------|-------------------------|--------------------------------|
| 1 | USB no detectado en Boot Menu | Probar puerto USB 2.0; reiniciar; verificar tecla correcta para Boot Menu |
| 1 | GRUB no muestra Windows (dual boot) | `sudo dnf install os-prober && sudo grub2-mkconfig -o /boot/grub2/grub.cfg` |
| 2 | VM no obtiene IP | `sudo virsh net-start default` en el host Fedora |
| 2 | PostgreSQL no acepta conexión | Verificar pg_hba.conf permite scram-sha-256; recargar: `sudo systemctl reload postgresql-16` |
| 3 | Pagila no carga completamente | Ejecutar scripts en orden: schema primero, luego data; verificar usuario tiene privilegios CREATEDB |
| 3 | Hardware insuficiente para VM | Usar sandbox online (Neon, Docker, pgexercises) como alternativa válida |

------------------------------------------------------------------------

#### Recursos de Apoyo

##### Documentación Esencial

- [Guía de instalación de Fedora](https://docs.fedoraproject.org/es/project/help/)
- [PostgreSQL para principiantes](https://www.postgresqltutorial.com/)
- [Repositorio Pagila](https://github.com/JamesRonsonOp/SQL_Training_with_Pagila)

##### Sandboxes Online (alternativa o complemento)

- [Neon.tech](https://neon.tech/) → PostgreSQL serverless gratuito
- [DB Fiddle](https://www.db-fiddle.com/) → Pruebas rápidas de SQL
- [PG Exercises](https://pgexercises.com/) → Ejercicios interactivos con retroalimentación

##### Soporte UAM Iztapalapa

- Foro de la UEA Bases de Datos: Grupo de Telegram
- Contacto del profesor: Dr. Jesús Zavala Ruiz [correo electrónico](mailto:jzr@xanum.uam.mx)
- Horarios de asesoría intensiva durante el track: Grupo de Telegram

------------------------------------------------------------------------

#### Evaluación del Track Intensivo

##### Criterios de Aprobación (Enfoque en Progreso)

| Criterio | Peso | Indicador de Cumplimiento |
|----------------|----------------|-----------------------------------------|
| Entorno local operativo | 40% | Fedora + VM con PostgreSQL accesible, o sandbox online configurado |
| Pagila cargada y consultable | 30% | Base de datos Pagila operativa con al menos 3 consultas ejecutadas exitosamente |
| Bitácora de aprendizaje | 20% | Documentación de decisiones, comandos clave y reflexión sobre conceptos de DBMS |
| Validación y respaldo | 10% | Script de respaldo funcional y checklist de validación ejecutado |

##### Entregables Consolidados

1.  **Carpeta de evidencias** (`~/uea-bases-de-datos/`) conteniendo:

    ```         
    ~/uea-bases-de-datos/
    ├── bitacoras/
    │   ├── lab01_fedora.md
    │   ├── lab02_03_vm_postgres.md
    │   └── lab04_pagila.md
    ├── scripts/
    │   ├── validar_entorno.sh
    │   └── respaldo_pagila.sh
    ├── conexion.txt          ### Parámetros de acceso a PostgreSQL
    └── README.md            ### Instrucciones para replicar el entorno
    ```

2.  **Reflexión final** (máximo 1 página) respondiendo:

    - ¿Qué es un DBMS después de haber instalado PostgreSQL y cargado Pagila?
    - ¿Qué ventajas y desafíos identificas entre ejecutar PostgreSQL localmente vs. en un sandbox online?
    - ¿Cómo aplicarías lo aprendido en un proyecto real de la licenciatura?

------------------------------------------------------------------------

#### Mensaje Final del Profesor

Estimados estudiantes,

En solo tres días han pasado de no tener experiencia con Linux a contar con un entorno funcional para explorar sistemas gestores de bases de datos. Este logro no radica en memorizar comandos, sino en desarrollar la capacidad de aprender haciendo, de resolver problemas técnicos con criterio y de documentar su proceso para replicarlo y mejorarlo.

El objetivo de este track no era convertirlos en administradores de bases de datos expertos, sino preparar el terreno para que comiencen a preguntar: ¿cómo se organizan los datos? ¿qué hace que una consulta sea eficiente? ¿cómo garantizamos que la información no se pierda?

Ahora que tienen Pagila cargada y herramientas como DBeaver o pgAdmin a su disposición, los invito a explorar, experimentar y, sobre todo, a compartir sus hallazgos con sus compañeros. La próxima fase de la UEA construirá sobre esta base para profundizar en modelado, consultas avanzadas y diseño de esquemas.

Recuerden: el entorno que prepararon es una herramienta. Su valor como futuros profesionales de la computación está en saber usarla para resolver problemas reales con ética, creatividad y rigor técnico.

¡Éxito en la exploración de DBMS!

Dr. Jesús Zavala Ruiz Profesor
*Nota: Este track está optimizado para aprendizaje acelerado. Si algún paso requiere más tiempo, priorice la comprensión sobre la velocidad. El sandbox online está disponible como alternativa válida para estudiantes con limitaciones de hardware.*
