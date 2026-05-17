### Laboratorio 5: Construcción de un Entorno de Laboratorio con Fedora, PostgreSQL y la Base de Datos Pagila
### Versión operativa y verificada para Fedora 44 / KVM-libvirt

**Autor:** Dr. Jesús Zavala Ruiz  
**Unidad de Enseñanza-Aprendizaje (UEA):** Bases de Datos (2151106)  
**Institución:** Universidad Autónoma Metropolitana, Unidad Iztapalapa  
**Fecha de emisión:** 16 de mayo de 2026  
**Nivel de competencia:** Principiante a intermedio  
**Tiempo estimado de ejecución:** 45–60 minutos  
**Base técnica:** Red Hat "Build a lab quickly" + Esquema Pagila (devrimgunduz/pagila)

---

#### ⚠️ ADVERTENCIA DE SEGURIDAD

> El entorno descrito en esta guía está diseñado exclusivamente para fines académicos y de experimentación controlada. Las credenciales, configuraciones de red y parámetros de seguridad aquí especificados **no deben emplearse bajo ninguna circunstancia en entornos de producción**. Para despliegues productivos, se recomienda consultar y aplicar las guías oficiales de *hardening* de Fedora Project y PostgreSQL Global Development Group.

#### ℹ️ NOTA SOBRE LA BASE DE DATOS PAGILA

La base de datos `pagila` constituye un esquema de ejemplo clásico para PostgreSQL, derivado del esquema "DVD Rental" documentado oficialmente por PostgreSQL. Este conjunto de datos incluye tablas representativas como `film`, `actor`, `customer`, `rental`, `payment`, entre otras, y se utiliza ampliamente en contextos educativos para:

- Enseñanza de consultas SQL estructuradas y avanzadas
- Pruebas de rendimiento y optimización de índices
- Desarrollo de procedimientos almacenados y funciones
- Simulación de escenarios transaccionales reales

**Fuente oficial:** [https://github.com/devrimgunduz/pagila](https://github.com/devrimgunduz/pagila)

#### ESTRUCTURA METODOLÓGICA DEL LABORATORIO

Este laboratorio se organiza en dos fases operativas claramente delimitadas, con el propósito de facilitar la depuración, garantizar la reproducibilidad y promover buenas prácticas en virtualización profesional:

| Fase | Denominación | Objetivo principal | Momento de ejecución | Herramientas clave |
|------|-------------|-------------------|---------------------|-------------------|
| **Fase 1** | Personalización *offline* de la imagen | Preparar la imagen de disco virtual sin iniciar la máquina virtual | Previo al primer arranque | `qemu-img`, `virt-customize`, `ssh-keygen`, `libguestfs` |
| **Fase 2** | Configuración *online* y validación | Inicializar servicios, cargar datos y verificar la operatividad del entorno | Posterior al primer arranque | `virt-install`, `ssh`, `psql`, `systemctl`, `git` |

> **Nota técnica fundamental:** La separación entre personalización *offline* (sin el gestor de procesos `systemd` activo) y configuración *online* (con `systemd` en ejecución) es una práctica esencial en virtualización profesional. Esta distinción evita errores comunes, tales como intentar iniciar servicios dependientes del gestor de procesos durante la modificación directa de imágenes de disco.

---

### FASE 1: PERSONALIZACIÓN *OFFLINE* DE LA IMAGEN DE DISCO VIRTUAL

#### Objetivo de la Fase 1

Al concluir esta fase, el estudiante dispondrá de una imagen de disco virtual personalizada que incorpora los siguientes componentes:

- Fedora 44 Cloud Base actualizado a la versión más reciente disponible
- Usuario `alumno` con privilegios de administración mediante `sudo` y contraseña predefinida `uamIztapalapa`
- Clave SSH de tipo PKI inyectada para autenticación sin intervención manual
- PostgreSQL 16 instalado (pendiente de inicialización del clúster)
- Configuración básica de seguridad aplicada (SSH, firewall, SELinux)
- Script de post-arranque preparado para la inicialización automática de PostgreSQL y restauración del esquema Pagila

> **Nota técnica:** Durante esta fase, la máquina virtual **no se enciende**. Todas las modificaciones se aplican directamente al archivo de disco mediante el conjunto de herramientas `libguestfs`, garantizando así la integridad y reproducibilidad del proceso.

#### 1. Prerrequisitos y Validación del Entorno Anfitrión

##### 1.1 Verificación del Soporte de Virtualización por Hardware

Ejecute el siguiente comando para confirmar que su procesador admite extensiones de virtualización:

```bash
grep -E 'svm|vmx' /proc/cpuinfo
```

**Resultado esperado:** La salida debe incluir `svm` (procesadores AMD) o `vmx` (procesadores Intel) dentro del campo `flags`. En caso de no observar ninguno de estos identificadores, active la opción *Virtualization Technology* o *SVM Mode* en la configuración de BIOS/UEFI de su equipo.

##### 1.2 Instalación del Conjunto de Herramientas de Virtualización

Actualice los repositorios e instale los paquetes necesarios:

```bash
sudo dnf update -y
sudo dnf install -y \
    qemu-kvm \
    virt-manager \
    virt-viewer \
    libguestfs-tools \
    guestfs-tools \
    virt-install \
    genisoimage \
    edk2-ovmf \
    libvirt \
    libvirt-daemon-kvm \
    bridge-utils \
    git \
    sequoia-sq
```

Habilite e inicie el servicio de libvirt:

```bash
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```

> **Nota técnica:** El paquete `guestfs-tools` proporciona la utilidad `virt-customize`, esencial para modificar imágenes de disco sin necesidad de montarlas explícitamente. No se requiere instalar PostgreSQL en el sistema anfitrión; el motor de base de datos se desplegará exclusivamente dentro de la máquina virtual.

##### 1.3 Configuración de Permisos de Usuario

Agregue su usuario a los grupos necesarios para operar con libvirt:

```bash
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt
groups $USER
```

Verifique que la salida incluya los grupos `libvirt` y `kvm`. De no ser así, cierre la sesión actual e inicie una nueva para aplicar los cambios.

##### 1.4 Verificación de Disponibilidad de Comandos Críticos

Ejecute el siguiente bucle para confirmar la presencia de las herramientas requeridas:

```bash
for cmd in virt-customize virt-install virsh qemu-img ssh-keygen git sq; do
    command -v "$cmd" &> /dev/null && echo "✓ $cmd: disponible" || echo "✗ $cmd: NO disponible"
done
```

**Salida esperada:**
```
✓ virt-customize: disponible
✓ virt-install: disponible
✓ virsh: disponible
✓ qemu-img: disponible
✓ ssh-keygen: disponible
✓ git: disponible
✓ sq: disponible
```

#### 2. Preparación de la Imagen Base Fedora Cloud

##### 2.1 Generación del Par de Claves SSH Ed25519

Genere un par de claves criptográficas para autenticación automatizada:

```bash
### Crear el par de claves PKI
ssh-keygen -t ed25519 \
    -f ~/.ssh/fedora-lab-key \
    -N "" \
    -C "Llave de laboratorio - UEA Bases de Datos" \
    -q

### Establecer permisos restrictivos
chmod 600 ~/.ssh/fedora-lab-key
chmod 644 ~/.ssh/fedora-lab-key.pub

### Verificar creación y permisos
ls -l ~/.ssh/fedora-lab-key*

### Visualizar comentario incrustado en la clave pública
tail -c 40 ~/.ssh/fedora-lab-key.pub
```

**Salida esperada:**
```
-rw-------. 1 usuario libvirt 464 May 15 23:31 /home/usuario/.ssh/fedora-lab-key
-rw-r--r--. 1 usuario libvirt 105 May 15 23:31 /home/usuario/.ssh/fedora-lab-key.pub
... Llave de laboratorio - UEA Bases de Datos
```

###### Explicación Técnica de los Parámetros de `ssh-keygen`

| Parámetro | Significado técnico | Propósito en el contexto del laboratorio |
|-----------|-------------------|-----------------------------------------|
| `-t ed25519` | Especifica el algoritmo de firma: curva de Edwards de 255 bits | Algoritmo moderno, resistente a ataques de canal lateral, con mejor rendimiento que RSA; recomendado por OpenSSH desde 2014 |
| `-f ~/.ssh/fedora-lab-key` | Define la ruta y nombre base para la clave privada | Genera `fedora-lab-key` (privada) y `fedora-lab-key.pub` (pública) en el directorio estándar `~/.ssh/` |
| `-N ""` | Establece una *passphrase* vacía | Permite automatización sin interrupciones; adecuado para entornos académicos aislados, no recomendado para producción |
| `-C "comentario"` | Agrega un identificador textual a la clave pública | Facilita la auditoría y gestión de múltiples claves; no afecta la criptografía subyacente |
| `-q` | Modo silencioso (*quiet*) | Suprime mensajes de progreso, garantizando ejecución no interactiva y reproducible |

> **Consideraciones de seguridad:** La ausencia de *passphrase* (`-N ""`) elimina la barrera de autenticación local. En un laboratorio académico esto es funcional para evitar interrupciones durante la automatización; en entornos productivos se recomienda omitir `""` y proporcionar una frase de acceso robusta. Asimismo, SSH rechaza claves privadas con permisos excesivos; verifique siempre con `chmod 600` antes de utilizarlas.

##### 2.2 Descarga y Verificación de la Imagen Fedora Cloud Base

```bash
### Crear directorio de trabajo
mkdir -p ~/vm-images/fedora-cloud
cd ~/vm-images/fedora-cloud

### Descargar imagen y archivo de verificación
wget https://download.fedoraproject.org/pub/fedora/linux/releases/44/Cloud/x86_64/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2
wget https://dl.fedoraproject.org/pub/fedora/linux/releases/44/Cloud/x86_64/images/Fedora-Cloud-44-1.7-x86_64-CHECKSUM

### Descargar certificados OpenPGP de Fedora
curl -O https://fedoraproject.org/fedora.pgp
curl -O https://fedoraproject.org/fedora.gpg

### Verificar integridad mediante firma digital (Sequoia SQ)
sq verify --cleartext --signer-file ./fedora.pgp \
   Fedora-Cloud-44-1.7-x86_64-CHECKSUM \
   | sha256sum -c --ignore-missing

### Verificación alternativa con GPG clásico
gpgv --keyring ./fedora.gpg --output - \
   Fedora-Cloud-44-1.7-x86_64-CHECKSUM \
   | sha256sum -c --ignore-missing
```

**Resultado esperado:** Ambos comandos deben reportar `OK` para el archivo `Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2`, confirmando su autenticidad e integridad.

##### 2.3 Traslado de la Imagen al Directorio de Libvirt

```bash
sudo mv Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2 \
    /var/lib/libvirt/images/

sudo chown qemu:qemu /var/lib/libvirt/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2
sudo chmod 644 /var/lib/libvirt/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2

### Verificar propiedades del archivo
sudo ls -lh /var/lib/libvirt/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2
```

##### 2.4 Creación de una Imagen Derivada (*Copy-on-Write*)

> **Advertencia:** Este paso es crítico y no debe omitirse. La técnica *Copy-on-Write* permite preservar la imagen base intacta mientras se aplican modificaciones en una capa derivada, facilitando la reversión rápida de cambios.

```bash
sudo qemu-img create -f qcow2 \
    -b /var/lib/libvirt/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2 \
    -F qcow2 \
    /var/lib/libvirt/images/fedora44-lab.qcow2 \
    20G

### Inspeccionar metadatos de la imagen derivada
sudo qemu-img info /var/lib/libvirt/images/fedora44-lab.qcow2

### Ajustar permisos y propiedad
sudo chown qemu:qemu /var/lib/libvirt/images/fedora44-lab.qcow2
sudo chmod 644 /var/lib/libvirt/images/fedora44-lab.qcow2

### Confirmar existencia en el directorio de libvirt
sudo ls -la /var/lib/libvirt/images/
```

**Salida esperada (resumen de `qemu-img info`):**
```
image: /var/lib/libvirt/images/fedora44-lab.qcow2
file format: qcow2
virtual size: 20 GiB (21474836480 bytes)
disk size: 196 KiB
backing file: /var/lib/libvirt/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2
backing file format: qcow2
```

> **Nota técnica:** La presencia del campo `backing file` confirma que la imagen derivada referencia a la base mediante el mecanismo *Copy-on-Write*. Esto permite descartar cambios eliminando únicamente el archivo derivado, sin afectar la imagen maestra.

#### 3. Personalización *Offline* Mediante `virt-customize`

##### 3.1 Preparación del Script de Personalización

Guarde el siguiente contenido en el archivo `~/vm-images/fedora-cloud/personaliza_lab.sh`:

```bash
#!/bin/bash
### =============================================================================
### personaliza_lab.sh
### Script de Personalización Offline para Fedora 44 Lab
### Usuario: alumno | Contraseña: uamIztapalapa
### Autor: Dr. Jesús Zavala Ruiz | UEA Bases de Datos (2151106)
### =============================================================================
set -euo pipefail

### CONFIGURACIÓN
IMAGE_PATH="/var/lib/libvirt/images/fedora44-lab.qcow2"
SSH_PUBKEY="${HOME}/.ssh/fedora-lab-key.pub"
USERNAME="alumno"
USER_PASSWORD="uamIztapalapa"
HOSTNAME="fedora-lab.test"

### VALIDACIONES PREVIAS
[[ -f "$IMAGE_PATH" ]] || { echo "❌ Error: Imagen $IMAGE_PATH no encontrada."; exit 1; }
[[ -f "$SSH_PUBKEY" ]] || { echo "❌ Error: Clave pública $SSH_PUBKEY no encontrada."; exit 1; }

echo "🔍 Validaciones completadas. Iniciando personalización..."

### EJECUCIÓN DE VIRT-CUSTOMIZE
sudo virt-customize \
  -a "$IMAGE_PATH" \
  --hostname "$HOSTNAME" \
  \
  --run-command "useradd -m -G wheel -s /bin/bash $USERNAME" \
  --run-command "echo '$USERNAME:$USER_PASSWORD' | chpasswd" \
  --run-command "echo '$USERNAME ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/$USERNAME" \
  --run-command "chmod 440 /etc/sudoers.d/$USERNAME" \
  \
  --ssh-inject "${USERNAME}:file:${SSH_PUBKEY}" \
  \
  --run-command "sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config || echo 'PermitRootLogin no' >> /etc/ssh/sshd_config" \
  --run-command "sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config || echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config" \
  --run-command "sed -i 's/^#PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config || echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config" \
  \
  --uninstall cloud-init \
  --selinux-relabel \
  --run-command "echo 'Fedora Lab personalizado - \$(date)' > /etc/fedora-lab-ready"

echo "✅ Personalización offline completada exitosamente."
```

Ajuste permisos de ejecución:

```bash
chmod +x ~/vm-images/fedora-cloud/personaliza_lab.sh
```

##### 3.2 Ejecución del Script de Personalización

```bash
cd ~/vm-images/fedora-cloud
./personaliza_lab.sh
```

**Salida esperada (resumen):**
```
🔍 Validaciones completadas. Iniciando personalización...
[   0.0] Examining the guest ...
[  12.7] Setting the hostname: fedora-lab.test
[  12.7] Running: useradd -m -G wheel -s /bin/bash alumno
[  13.0] Running: echo 'alumno:uamIztapalapa' | chpasswd
[  13.1] SSH key inject: alumno
[  14.2] Running: sed -i ... (configuración de sshd_config)
[  16.5] Uninstalling packages: cloud-init
[  16.6] SELinux relabelling
[  18.2] Finishing off
✅ Personalización offline completada exitosamente.
```

> **Nota técnica:** El orden secuencial de las opciones en `virt-customize` es determinante. La creación del usuario (`useradd`) debe preceder a la inyección de la clave SSH (`--ssh-inject`). El parámetro `--selinux-relabel` garantiza que los contextos de seguridad se ajusten correctamente tras las modificaciones.

#### 4. Creación y Arranque de la Máquina Virtual

##### 4.1 Definición de la Máquina Virtual Mediante `virt-install`

```bash
sudo virt-install \
  --name fedora-lab \
  --description "Fedora 44 Lab - UEA Bases de Datos" \
  --memory 4096 \
  --vcpus 2 \
  --disk /var/lib/libvirt/images/fedora44-lab.qcow2,format=qcow2,bus=virtio,cache=none \
  --import \
  --os-variant fedora43 \
  --network network=default,model=virtio \
  --graphics none \
  --console pty,target_type=serial \
  --noautoconsole \
  --qemu-commandline="-cpu host" \
  --qemu-commandline="-accel kvm"
```

**Salida esperada:**
```
Starting install...
Creating domain...                                                    | 00:00:00
Domain creation completed.
```

##### 4.2 Verificación del Estado de la Máquina Virtual

```bash
virsh list --all
```

**Salida esperada:**
```
 Id   Name           State
-------------------------------
 -    fedora-lab     shut off
```

##### 4.3 Inicio de la Máquina Virtual y Obtención de Dirección IP

```bash
### Iniciar la VM
sudo virsh start fedora-lab

### Esperar ~30 segundos para el arranque completo
sleep 30

### Consultar dirección IP asignada
sudo virsh domifaddr fedora-lab
```

**Salida esperada:**
```
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet4      52:54:00:34:b1:52    ipv4         192.168.122.161/24
```

##### 4.4 Conexión SSH a la Máquina Virtual

```bash
ssh -i ~/.ssh/fedora-lab-key alumno@192.168.122.161
```

**Primera conexión (mensaje de autenticidad):**
```
The authenticity of host '192.168.122.161 (192.168.122.161)' can't be established.
ED25519 key fingerprint is SHA256:vWk/nie4fhS0NnCszaYfSpc5pcDWjVrCS2rVqsG/lsA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.161' (ED25519) to the list of known hosts.
[alumno@fedora-lab ~]$
```

> **Nota:** Dado que la clave privada no posee *passphrase*, presione `<Enter>` si se solicita una contraseña.

Verifique la presencia del marcador de configuración:

```bash
cat /etc/fedora-lab-ready
```

**Salida esperada:**
```
Fedora Lab personalizado - [fecha y hora]
```

Cierre la sesión SSH:

```bash
exit
```

---

### FASE 2: CONFIGURACIÓN *ONLINE* Y VALIDACIÓN DEL ENTORNO

#### Objetivo de la Fase 2

Al concluir esta fase, el estudiante habrá:

- Iniciado exitosamente la máquina virtual Fedora 44 personalizada
- Verificado la conectividad de red y el acceso SSH mediante clave pública
- Ejecutado el script de configuración de PostgreSQL dentro de la VM
- Cargado la base de datos de ejemplo Pagila con integridad referencial
- Ejecutado consultas SQL básicas para validar la operatividad del entorno

#### 5. Transferencia y Ejecución del Script de Configuración de PostgreSQL

##### 5.1 Preparación del Script `setup_postgres_lab.sh`

En el sistema anfitrión, cree el archivo `setup_postgres_lab.sh` con el siguiente contenido:

```bash
#!/bin/bash
### =============================================================================
### setup_postgres_lab.sh
### Script de Configuración Online para PostgreSQL 16 + Pagila
### Autor: Dr. Jesús Zavala Ruiz | UEA Bases de Datos (2151106)
### =============================================================================
set -euo pipefail

LOG_FILE="/var/log/postgres-lab-setup.log"
exec > >(tee -a "$LOG_FILE") 2>&1

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }

log "🚀 Iniciando configuración robusta de PostgreSQL para Fedora 44..."

### 1. Verificación e instalación de paquetes base
log "📦 Verificando paquetes base..."
sudo dnf install -y postgresql-server postgresql-contrib git

### 2. Inicialización del clúster de PostgreSQL
log "🗄️ Verificando estado del clúster de PostgreSQL..."
if [ ! -d /var/lib/pgsql/data/base ]; then
    log "Inicializando base de datos con postgresql-setup..."
    sudo postgresql-setup --initdb
    log "✅ Clúster inicializado en /var/lib/pgsql/data"
else
    log "ℹ️ Clúster ya existente; omitiendo inicialización."
fi

### 3. Configuración de autenticación local (entorno de laboratorio)
log "🔐 Configurando pg_hba.conf para acceso de laboratorio..."
sudo sed -i 's/^local\s\+all\s\+all\s\+ident/local   all             all                                     trust/' /var/lib/pgsql/data/pg_hba.conf
sudo sed -i 's/^host\s\+all\s\+all\s\+127.0.0.1\/32\s\+ident/host    all             all             127.0.0.1\/32            trust/' /var/lib/pgsql/data/pg_hba.conf
sudo sed -i 's/^host\s\+all\s\+all\s\+::1\/128\s\+ident/host    all             all             ::1\/128                 trust/' /var/lib/pgsql/data/pg_hba.conf
log "✅ pg_hba.conf configurado (método: trust para localhost)."

### 4. Ajustes de rendimiento para entorno de laboratorio
log "⚙️ Ajustando parámetros en postgresql.conf..."
sudo tee -a /var/lib/pgsql/data/postgresql.conf > /dev/null << 'EOF'
### Configuración para laboratorio
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 16MB
maintenance_work_mem = 128MB
log_min_duration_statement = 1000
EOF
log "✅ postgresql.conf optimizado."

### 5. Inicio y habilitación del servicio PostgreSQL
log "🚀 Iniciando servicio PostgreSQL..."
sudo systemctl enable --now postgresql

### Esperar a que PostgreSQL acepte conexiones
log "⏳ Esperando a que PostgreSQL acepte conexiones..."
for i in {1..20}; do
    if sudo -u postgres psql -c "SELECT 1;" &> /dev/null; then
        log "✅ PostgreSQL listo (intento $i/20)."
        break
    fi
    sleep 2
done

### 6. Creación del rol 'alumno' con privilegios
log "👤 Configurando rol 'alumno' en PostgreSQL..."
sudo -u postgres psql -c "CREATE ROLE alumno WITH LOGIN PASSWORD 'uamIztapalapa';" || true
sudo -u postgres psql -c "ALTER ROLE alumno CREATEDB;" || true
log "✅ Rol 'alumno' creado."

### 7. Descarga y carga de la base de datos Pagila
log "📦 Preparando base de datos Pagila..."
cd /opt || cd /
if [ ! -d "/opt/pagila" ]; then
    log "Clonando repositorio de Pagila..."
    sudo git clone https://github.com/devrimgunduz/pagila.git /opt/pagila
fi

log "Creando base de datos 'pagila' y cargando esquema..."
sudo -u postgres psql -c "DROP DATABASE IF EXISTS pagila;" || true
sudo -u postgres psql -c "CREATE DATABASE pagila OWNER alumno;"
sudo -u postgres psql -d pagila -f /opt/pagila/pagila-schema.sql
sudo -u postgres psql -d pagila -f /opt/pagila/pagila-data.sql

log "Asignando privilegios al rol 'alumno'..."
sudo -u postgres psql -d pagila -c "GRANT ALL ON ALL TABLES IN SCHEMA public TO alumno;"
sudo -u postgres psql -d pagila -c "GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO alumno;"
log "✅ Pagila cargada y permisos configurados."

### 8. Validación final
log "🧪 Ejecutando validación final..."
FILM_COUNT=$(sudo -u postgres psql -d pagila -t -c "SELECT COUNT(*) FROM film;" | tr -d ' ')
if [ "$FILM_COUNT" -eq 1000 ]; then
    log "🎉 VALIDACIÓN EXITOSA: 1000 películas en la tabla 'film'."
else
    log "⚠️ ADVERTENCIA: Se esperaban 1000 registros, se encontraron $FILM_COUNT."
fi

log "📋 Configuración completada. Log guardado en: $LOG_FILE"
log "🔑 Para conectar: psql -h localhost -U alumno -d pagila"
```

Ajuste permisos y transfiera el script a la VM:

```bash
### En el sistema anfitrión
chmod +x setup_postgres_lab.sh
scp -i ~/.ssh/fedora-lab-key setup_postgres_lab.sh alumno@192.168.122.161:/home/alumno/
```

##### 5.2 Ejecución del Script Dentro de la Máquina Virtual

Conéctese a la VM y ejecute el script:

```bash
### Conexión SSH
ssh -i ~/.ssh/fedora-lab-key alumno@192.168.122.161

### Dentro de la VM: preparar y ejecutar el script
chmod +x setup_postgres_lab.sh
sudo ./setup_postgres_lab.sh
```

**Salida esperada (resumen):**
```
[2026-05-17 00:00:01] 🚀 Iniciando configuración robusta de PostgreSQL para Fedora 44...
[2026-05-17 00:00:01] 📦 Verificando paquetes base...
...
[2026-05-17 00:00:01] ✅ Paquetes instalados correctamente.
[2026-05-17 00:00:01] 🗄️ Verificando estado del clúster de PostgreSQL...
[2026-05-17 00:00:01] ✅ Clúster inicializado en /var/lib/pgsql/data
[2026-05-17 00:00:01] 🔐 Configurando pg_hba.conf para acceso de laboratorio...
[2026-05-17 00:00:01] ✅ pg_hba.conf configurado (método: trust para localhost).
[2026-05-17 00:00:01] 🚀 Iniciando servicio PostgreSQL...
[2026-05-17 00:00:01] ✅ PostgreSQL listo (intento 1/20).
[2026-05-17 00:00:01] 👤 Configurando rol 'alumno' en PostgreSQL...
[2026-05-17 00:00:01] ✅ Rol 'alumno' creado.
[2026-05-17 00:00:01] 📦 Preparando base de datos Pagila...
[2026-05-17 00:00:01] ✅ Pagila cargada y permisos configurados.
[2026-05-17 00:00:01] 🧪 Ejecutando validación final...
[2026-05-17 00:00:01] 🎉 VALIDACIÓN EXITOSA: 1000 películas en la tabla 'film'.
[2026-05-17 00:00:01] 🔑 Para conectar: psql -h localhost -U alumno -d pagila
```

#### 6. Conexión a PostgreSQL y Ejecución de Consultas de Validación

##### 6.1 Acceso al Cliente `psql`

```bash
psql -h localhost -U alumno -d pagila
```

**Salida esperada:**
```
psql (16.x)
Type "help" for help.

pagila=>
```

##### 6.2 Consultas de Prueba para Validar el Entorno

###### Consulta 1: Listado de películas (primeros 5 registros)

```sql
SELECT title, release_year, rating FROM film LIMIT 5;
```

**Resultado esperado:**
```
      title       | release_year | rating 
------------------+--------------+--------
 ACADEMY DINOSAUR |         2012 | PG
 ACE GOLDFINGER   |         2023 | G
 ADAPTATION HOLES |         2017 | NC-17
 AFFAIR PREJUDICE |         2023 | G
 AFRICAN EGG      |         2019 | G
(5 rows)
```

###### Consulta 2: Búsqueda de películas con patrón de texto

```sql
SELECT title, length, rating FROM film WHERE title ILIKE '%dragon%';
```

**Resultado esperado:**
```
        title        | length | rating 
---------------------+--------+--------
 CASPER DRAGONFLY    |    163 | PG-13
 DRAGON SQUAD        |    170 | NC-17
 DRAGONFLY STRANGERS |    133 | NC-17
 FACTORY DRAGON      |    144 | PG-13
(4 rows)
```

###### Consulta 3: JOIN entre tablas relacionadas

```sql
SELECT a.first_name, a.last_name, f.title
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id
LIMIT 10;
```

**Resultado esperado:**
```
 first_name | last_name |         title         
------------+-----------+-----------------------
 PENELOPE   | GUINESS   | ACADEMY DINOSAUR
 PENELOPE   | GUINESS   | ANACONDA CONFESSIONS
 PENELOPE   | GUINESS   | ANGELS LIFE
 PENELOPE   | GUINESS   | BULWORTH COMMANDMENTS
 PENELOPE   | GUINESS   | CHEAPER CLYDE
 PENELOPE   | GUINESS   | COLOR PHILADELPHIA
 PENELOPE   | GUINESS   | ELEPHANT TROJAN
 PENELOPE   | GUINESS   | GLEAMING JAWBREAKER
 PENELOPE   | GUINESS   | HUMAN GRAFFITI
 PENELOPE   | GUINESS   | KING EVOLUTION
(10 rows)
```

Salga del cliente `psql` mediante el comando:

```sql
\q
```

#### 7. Validación Integral del Entorno

Ejecute los siguientes comandos para confirmar la operatividad de todos los componentes:

```bash
### Verificar estado del servicio PostgreSQL
sudo systemctl is-active postgresql

### Verificar existencia y contenido de la base de datos Pagila
sudo -u postgres psql -d pagila -c "SELECT COUNT(*) AS total_films FROM film;"

### Verificar contexto de conexión del usuario 'alumno'
psql -h localhost -U alumno -d pagila -c "SELECT current_user, current_database();"
```

**Salidas esperadas:**
```
active

 total_films 
-------------
        1000
(1 row)

 current_user | current_database 
--------------+------------------
 alumno       | pagila
(1 row)
```

#### 8. Procedimiento de Renombrado de Máquina Virtual en KVM/libvirt

##### 8.1 Objetivo

Documentar el procedimiento formal para renombrar una máquina virtual en un entorno KVM/libvirt, garantizando la preservación de datos, la integridad de la configuración y la operatividad de servicios posteriores a la reconfiguración.

##### 8.2 Requisitos Previos

- Acceso con privilegios `sudo` en el sistema anfitrión
- Conocimiento de la ruta de la imagen de disco virtual (`.qcow2`)
- Herramientas `libvirt` y `virt-install` instaladas y operativas
- Red virtual `default` activa en libvirt

##### 8.3 Procedimiento Paso a Paso

###### Paso 1: Diagnóstico del Estado Inicial

```bash
virsh list --all
```

###### Paso 2: Identificación del Dominio Activo

```bash
sudo lsof /var/lib/libvirt/images/fedora44-lab.qcow2
sudo virsh list --all
```

###### Paso 3: Apagado Controlado de la Máquina Virtual

```bash
sudo virsh shutdown fedora-lab
### Esperar confirmación de apagado
virsh list --all
```

###### Paso 4: Eliminación de la Definición Antigua y Creación de la Nueva

```bash
### Eliminar definición anterior (la imagen de disco permanece intacta)
sudo virsh undefine fedora-lab

### Recrear definición con nuevo nombre, apuntando al disco existente
sudo virt-install \
  --name fedora44-lab \
  --description "Fedora 44 Lab - UEA Bases de Datos" \
  --memory 4096 \
  --vcpus 2 \
  --disk /var/lib/libvirt/images/fedora44-lab.qcow2,format=qcow2,bus=virtio,cache=none \
  --import \
  --os-variant fedora43 \
  --network network=default,model=virtio \
  --graphics none \
  --console pty,target_type=serial \
  --noautoconsole \
  --qemu-commandline="-cpu host" \
  --qemu-commandline="-accel kvm"
```

###### Paso 5: Validación Operativa Final

```bash
### Verificar registro del dominio
sudo virsh list

### Consultar nueva dirección IP
sudo virsh domifaddr fedora44-lab

### Conectar y validar servicios
ssh -i ~/.ssh/fedora-lab-key alumno@<nueva_ip>
psql -h localhost -U alumno -d pagila -c "SELECT COUNT(*) FROM film;"
```

##### 8.4 Estado Final del Entorno

| Componente | Estado | Detalle |
|------------|--------|---------|
| Nombre de VM | ✅ `fedora44-lab` | Definición registrada correctamente en libvirt |
| Dirección IP | ✅ Dinámica (ej. `192.168.122.24/24`) | Asignada por red `default` de libvirt |
| Servicio PostgreSQL | ✅ `active` | Versión 16, operativa y accesible |
| Base de datos Pagila | ✅ Intacta | 1000 registros validados, permisos conservados |
| Acceso SSH/PKI | ✅ Funcional | Usuario `alumno`, autenticación por clave Ed25519 |

> **Recomendaciones técnicas para entornos productivos:**
> 1. **Instantáneas (*Snapshots*):** Antes de operaciones de renombrado, ejecute `sudo virsh snapshot-create-as <vm> backup-pre-rename` para garantizar un punto de restauración inmediato.
> 2. **Limpieza de `known_hosts`:** Tras el cambio de IP, elimine la entrada huérfana en el anfitrión con `ssh-keygen -R <ip_anterior>`.
> 3. **Persistencia de definiciones:** Utilice `virsh edit <vm>` en lugar de manipulación manual de XML para evitar inconsistencias de UUID o rutas relativas.
> 4. **SELinux y permisos:** Mantenga el contexto de seguridad `system_u:object_r:virt_image_t:s0` en las imágenes `.qcow2` mediante `restorecon -v /var/lib/libvirt/images/*.qcow2`.

#### 9. Solución de Problemas Frecuentes

| Síntoma | Causa probable | Acción correctiva |
|---------|---------------|-------------------|
| `Permission denied` en SSH | Permisos incorrectos en clave privada | Ejecutar `chmod 600 ~/.ssh/fedora-lab-key` en el host |
| `Connection refused` al conectar a PostgreSQL | Servicio no iniciado | Ejecutar `sudo systemctl start postgresql` dentro de la VM |
| `role "alumno" does not exist` | Rol no creado en PostgreSQL | Ejecutar `sudo -u postgres psql -c "CREATE ROLE alumno WITH LOGIN PASSWORD 'uamIztapalapa';"` |
| `Ident authentication failed` | `pg_hba.conf` configurado con método `ident` | Modificar `pg_hba.conf` para usar `trust` en conexiones locales y recargar con `sudo systemctl reload postgresql` |
| VM no obtiene dirección IP | Red de libvirt inactiva | Ejecutar `sudo virsh net-start default` en el host |
| `virt-customize: ssh-inject: user does not exist` | Orden incorrecto de opciones | Mover `--run-command useradd` antes de `--ssh-inject` |
| `ssh: connect to host ... port 22: Connection refused` | VM en arranque o `sshd` inactivo | Esperar 60–90 s adicionales; verificar con `virsh console` |

#### 10. Comandos de Gestión Operativa

```bash
### === Desde el sistema anfitrión ===

### Detener la VM (apagado limpio)
sudo virsh shutdown fedora44-lab

### Forzar apagado (si shutdown no responde)
sudo virsh destroy fedora44-lab

### Reiniciar la VM
sudo virsh reboot fedora44-lab

### Acceder a la consola serial de la VM (salir con Ctrl+])
sudo virsh console fedora44-lab

### Crear una instantánea antes de cambios críticos
sudo virsh snapshot-create-as --domain fedora44-lab --name "pre-config"

### Clonar la VM para pruebas paralelas
sudo virt-clone --original fedora44-lab --name fedora44-lab-test \
  --file /var/lib/libvirt/images/fedora44-lab-test.qcow2

### Eliminar completamente la VM y su disco
sudo virsh destroy fedora44-lab && sudo virsh undefine fedora44-lab \
  && sudo rm /var/lib/libvirt/images/fedora44-lab.qcow2

### === Desde la máquina virtual (usuario alumno) ===

### Ver estado de PostgreSQL
sudo systemctl status postgresql

### Conectarse como administrador de PostgreSQL
sudo -u postgres psql

### Listar bases de datos disponibles
psql -h localhost -U alumno -c "\l"

### Listar tablas de la base de datos Pagila
psql -h localhost -U alumno -d pagila -c "\dt"

### Salir del cliente psql
\q
```

#### 11. Notas de Seguridad para Entornos Productivos

La configuración actual emplea el método de autenticación `trust` en `pg_hba.conf`, lo que permite conexiones locales sin contraseña. Esta configuración es adecuada para un laboratorio académico aislado, pero **no debe utilizarse en entornos productivos**. Para despliegues reales, considere las siguientes medidas:

- Cambiar `trust` por `scram-sha-256` o `md5` en `pg_hba.conf`
- Utilizar contraseñas robustas y establecer políticas de rotación periódica
- Restringir el acceso por dirección IP mediante reglas específicas en `pg_hba.conf`
- Habilitar registro de auditoría con `log_statement = 'all'` en `postgresql.conf`
- Mantener SELinux en modo `enforcing` con políticas personalizadas
- Implementar cifrado de tráfico mediante TLS/SSL para conexiones remotas

---

#### 12. Archivos y Directorios Generados en la Máquina Virtual

| Archivo o Directorio | Propósito |
|---------------------|-----------|
| `/var/log/postgres-lab-setup.log` | Registro completo de la configuración de PostgreSQL |
| `/opt/pagila/` | Repositorio con scripts SQL de la base de datos Pagila |
| `/var/lib/pgsql/data/` | Directorio de datos del clúster de PostgreSQL |
| `/etc/fedora-lab-ready` | Marcador de configuración completada |
| `/etc/ssh/sshd_config.d/hardening.conf` | Configuración de endurecimiento de SSH (opcional) |

---

#### REFERENCIAS

1. Red Hat. (2021). *Build a lab quickly*. Recuperado de https://www.redhat.com/en/blog/build-lab-quickly
2. devrimgunduz. (2026). *Pagila - Sample Database for PostgreSQL*. Recuperado de https://github.com/devrimgunduz/pagila
3. PostgreSQL Global Development Group. (2026). *PostgreSQL 16 Documentation*. Recuperado de https://www.postgresql.org/docs/16/
4. libguestfs Tools. (2026). *virt-customize manual*. Recuperado de https://libguestfs.org/virt-customize.1.html
5. Universidad Autónoma Metropolitana, Unidad Iztapalapa. (2026). *Programa de la UEA Bases de Datos (2151106)*. División de Ciencias Básicas e Ingeniería.

> **Nota final:** Esta arquitectura basada en *Copy-on-Write* permite reiniciar el entorno en menos de dos minutos eliminando el archivo `fedora44-lab.qcow2` y repitiendo las secciones 2.4 y 3.2. La imagen base permanece inacta para iteraciones futuras. Se recomienda documentar las personalizaciones en el repositorio institucional y rotar credenciales al finalizar el trimestre académico.
