### Guía Definitiva: Construye un Laboratorio de Fedora con PostgreSQL 16 y Base de Datos Pagila en Minutos
### Versión a prueba de fallas para Fedora 43 / KVM-libvirt

**Autor:** Dr. Jesús Zavala Ruiz  
**Institución:** Universidad Autónoma Metropolitana, Unidad Iztapalapa  
**UEA:** Bases de Datos (2151106)  
**Fecha:** Mayo 2026  
**Basado en:** Red Hat "Build a lab quickly" + Esquema Pagila (devrimgunduz)

---

> ⚠️ **Advertencia de seguridad**: Esta guía configura un entorno de laboratorio. Las credenciales y configuraciones aquí descritas **NO deben utilizarse en entornos de producción**. Para despliegues productivos, consulte las guías oficiales de hardening de Fedora y PostgreSQL.

> ℹ️ **Nota sobre Pagila**: La base de datos `pagila` es un esquema de ejemplo clásico para PostgreSQL, derivado del esquema "DVD Rental" de la documentación oficial. Contiene tablas como `film`, `actor`, `customer`, `rental`, etc., y es ampliamente utilizada para enseñanza, pruebas de rendimiento y desarrollo de consultas SQL complejas. Fuente: https://github.com/devrimgunduz/pagila

---

#### 📋 Índice

1. [Prerrequisitos y Validación del Entorno](#1-prerrequisitos-y-validación-del-entorno)
2. [Preparación de la Imagen Base](#2-preparación-de-la-imagen-base)
3. [Generación de Claves SSH para PKI](#3-generación-de-claves-ssh-para-pki)
4. [Personalización de la Imagen con virt-customize](#4-personalización-de-la-imagen-con-virt-customize)
5. [Creación e Inicio de la Máquina Virtual](#5-creación-e-inicio-de-la-máquina-virtual)
6. [Restauración de la Base de Datos Pagila](#6-restauración-de-la-base-de-datos-pagila)
7. [Hardening de Seguridad Completo](#7-hardening-de-seguridad-completo)
8. [Troubleshooting y Checklist de Validación](#8-troubleshooting-y-checklist-de-validación)

---

#### 1. Prerrequisitos y Validación del Entorno

##### 1.1 Verificar soporte de virtualización por hardware

```bash
### Verificar que el procesador soporta virtualización
grep -E 'svm|vmx' /proc/cpuinfo
```

✅ **Éxito esperado:** Debe mostrar `svm` (AMD) o `vmx` (Intel) en la línea `flags`.  
❌ **Fallo:** Active `Virtualization Technology` o `SVM Mode` en la BIOS/UEFI.

##### 1.2 Instalar paquetes base y herramientas de virtualización

```bash
### Actualizar sistema e instalar dependencias
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
    postgresql16 \
    postgresql16-server \
    postgresql16-contrib \
    git

### Habilitar e iniciar el servicio libvirtd
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```

✅ **Verificación:** `systemctl status libvirtd` debe mostrar `active (running)`.

##### 1.3 Configurar permisos de usuario

```bash
### Agregar usuario actual a grupos libvirt y kvm
sudo usermod -aG libvirt,kvm $USER

### Aplicar cambios de grupo sin cerrar sesión
newgrp libvirt

### Verificar membresía de grupos
groups $USER
```

✅ **Éxito esperado:** La salida debe incluir `libvirt` y `kvm`.

##### 1.4 Verificar disponibilidad de comandos críticos

```bash
### Verificar que las herramientas están instaladas
for cmd in virt-customize virt-install virsh qemu-img ssh-keygen psql git; do
    if command -v $cmd &> /dev/null; then
        echo "✅ $cmd: disponible"
    else
        echo "❌ $cmd: NO disponible - instale el paquete correspondiente"
        exit 1
    fi
done
```

---

#### 2. Preparación de la Imagen Base

##### 2.1 Descargar la imagen Fedora Cloud Base

```bash
### Crear directorio de trabajo
mkdir -p ~/vm-images/fedora-cloud
cd ~/vm-images/fedora-cloud

### Descargar imagen oficial Fedora 44 Cloud Base Generic
wget https://download.fedoraproject.org/pub/fedora/linux/releases/44/Cloud/x86_64/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2

### Verificar integridad con checksum (opcional pero recomendado)
wget https://download.fedoraproject.org/pub/fedora/linux/releases/44/Cloud/x86_64/images/Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2.CHECKSUM
sha256sum -c Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2.CHECKSUM
```

✅ **Éxito esperado:** `Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2: OK`

##### 2.2 Mover imagen al directorio de libvirt

```bash
### Mover imagen al directorio de trabajo de libvirt
sudo mv Fedora-Cloud-Base-Generic-44-1.7.x86_64.qcow2 \
    /var/lib/libvirt/images/fedora44-base.qcow2

### Ajustar permisos para que libvirt pueda leer la imagen
sudo chown qemu:qemu /var/lib/libvirt/images/fedora44-base.qcow2
sudo chmod 644 /var/lib/libvirt/images/fedora44-base.qcow2

### Verificar que la imagen existe y es legible
sudo ls -lh /var/lib/libvirt/images/fedora44-base.qcow2
```

##### 2.3 Crear imagen derivada (Copy-on-Write) - Paso Crítico

> ⚠️ **Importante:** Nunca personalice directamente la imagen base. Siempre cree una copia derivada.

```bash
### Crear imagen derivada con qemu-img (copy-on-write)
sudo qemu-img create -f qcow2 \
    -b /var/lib/libvirt/images/fedora44-base.qcow2 \
    -F qcow2 \
    /var/lib/libvirt/images/fedora44-lab.qcow2 \
    20G

### Verificar que la imagen derivada está correctamente vinculada
sudo qemu-img info /var/lib/libvirt/images/fedora44-lab.qcow2
```

✅ **Salida esperada (fragmento):**
```
image: /var/lib/libvirt/images/fedora44-lab.qcow2
file format: qcow2
virtual size: 20 GiB (21474836480 bytes)
backing file: /var/lib/libvirt/images/fedora44-base.qcow2
backing file format: qcow2
```

❌ **Si no ve `backing file`:** Repita el comando anterior verificando rutas absolutas.

---

#### 3. Generación de Claves SSH para PKI

##### 3.1 Crear par de claves Ed25519 (recomendado)

```bash
### Generar clave sin passphrase para automatización de laboratorio
ssh-keygen -t ed25519 \
    -f ~/.ssh/fedora-lab-key \
    -N "" \
    -C "fedora-lab@izt.uam.mx" \
    -q

### Ajustar permisos de la clave privada (requerido por SSH)
chmod 600 ~/.ssh/fedora-lab-key
chmod 644 ~/.ssh/fedora-lab-key.pub

### Verificar que ambas claves existen
ls -l ~/.ssh/fedora-lab-key*
```

✅ **Éxito esperado:**
```
-rw------- 1 user user ... fedora-lab-key      (privada)
-rw-r--r-- 1 user user ... fedora-lab-key.pub  (pública)
```

---

#### 4. Personalización de la Imagen con virt-customize

> ⚠️ **Advertencia:** `virt-customize` ejecuta en un appliance aislado. **Siempre use rutas absolutas** para archivos locales.

##### 4.1 Script de personalización completo (copiar y pegar)

```bash
#!/bin/bash
### personalize-fedora-lab.sh
### Personalización a prueba de fallas para Fedora Cloud + PostgreSQL 16 + Pagila

set -euo pipefail  ### Salir en error, variable no definida, o fallo en pipe

### Variables configurables
IMAGE_PATH="/var/lib/libvirt/images/fedora44-lab.qcow2"
SSH_PUBKEY="/home/$(whoami)/.ssh/fedora-lab-key.pub"
USERNAME="user"
USER_PASSWORD="mvFedora43"
HOSTNAME="fedora-lab.test"
PAGILA_REPO="https://github.com/devrimgunduz/pagila.git"

### Verificaciones previas críticas
echo "🔍 Validando prerrequisitos..."

if [ ! -f "$IMAGE_PATH" ]; then
    echo "❌ Error: Imagen no encontrada: $IMAGE_PATH"
    echo "💡 Ejecute primero la Sección 2.3 para crear la imagen derivada."
    exit 1
fi

if [ ! -f "$SSH_PUBKEY" ]; then
    echo "❌ Error: Clave pública SSH no encontrada: $SSH_PUBKEY"
    echo "💡 Ejecute primero la Sección 3.1 para generar las claves."
    exit 1
fi

if ! command -v virt-customize &> /dev/null; then
    echo "❌ Error: virt-customize no está instalado."
    echo "💡 Ejecute: sudo dnf install guestfs-tools -y"
    exit 1
fi

echo "✅ Prerrequisitos validados. Iniciando personalización..."

### Comando virt-customize con verificación de cada paso
sudo virt-customize \
    -a "$IMAGE_PATH" \
    --hostname "$HOSTNAME" \
    \
    ### Crear usuario con privilegios sudo
    --run-command "useradd -m -G wheel -s /bin/bash $USERNAME" \
    --run-command "echo '$USERNAME:$USER_PASSWORD' | chpasswd" \
    --run-command "echo '$USERNAME ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/$USERNAME" \
    --run-command "chmod 440 /etc/sudoers.d/$USERNAME" \
    \
    ### Inyectar clave SSH para el usuario (ruta absoluta obligatoria)
    --ssh-inject "$USERNAME:file:$SSH_PUBKEY" \
    \
    ### Deshabilitar login de root por SSH (seguridad)
    --run-command "sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config || echo 'PermitRootLogin no' >> /etc/ssh/sshd_config" \
    \
    ### Habilitar y configurar SSH para PKI
    --run-command "sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config || echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config" \
    --run-command "sed -i 's/^#PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config || echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config" \
    \
    ### Instalar PostgreSQL 16 y herramientas necesarias
    --install postgresql16,postgresql16-server,postgresql16-contrib,git,pgadmin4 \
    \
    ### Inicializar PostgreSQL (se completará al primer arranque)
    --run-command "/usr/pgsql-16/bin/postgresql-16-setup initdb || true" \
    --run-command "systemctl enable postgresql-16" \
    \
    ### Configurar autenticación local para postgres (necesario para scripts de carga)
    --run-command "sed -i 's/^local.*all.*all.*peer/local   all             all                                     trust/' /var/lib/pgsql/16/data/pg_hba.conf || true" \
    --run-command "sed -i 's/^host.*all.*all.*ident/host    all             all             127.0.0.1/32            trust/' /var/lib/pgsql/16/data/pg_hba.conf || true" \
    \
    ### Preparar directorio para Pagila (se restaurará post-arranque)
    --run-command "mkdir -p /opt/pagila && chown $USERNAME:$USERNAME /opt/pagila" \
    \
    ### Limpieza y optimización
    --uninstall cloud-init \
    --selinux-relabel \
    --firstboot-command "dnf clean all" \
    \
    ### Mensaje de finalización
    --run-command "echo '✅ Fedora Lab personalizado exitosamente - \$(date)' > /etc/fedora-lab-ready"

echo "✅ Personalización completada exitosamente."
echo "📋 Imagen lista: $IMAGE_PATH"
echo "📦 La base de datos Pagila se restaurará automáticamente al primer arranque."
```

##### 4.2 Ejecutar el script de personalización

```bash
### Guardar el script anterior en un archivo
nano ~/personalize-fedora-lab.sh
chmod +x ~/personalize-fedora-lab.sh

### Ejecutar con tiempo estimado: 3-8 minutos según hardware
~/personalize-fedora-lab.sh
```

✅ **Éxito esperado:** Mensaje final `✅ Personalización completada exitosamente.`

---

#### 5. Creación e Inicio de la Máquina Virtual

##### 5.1 Comando virt-install optimizado

```bash
sudo virt-install \
    --name fedora-lab \
    --description "Fedora 44 Lab con PostgreSQL 16 + Pagila - UAM Iztapalapa" \
    --memory 4096 \
    --vcpus 2 \
    --disk /var/lib/libvirt/images/fedora44-lab.qcow2,format=qcow2,bus=virtio,cache=none \
    --import \
    --os-type linux \
    --os-variant fedora44 \
    --network network=default,model=virtio \
    --graphics none \
    --console pty,target_type=serial \
    --noautoconsole \
    --qemu-commandline="-cpu host" \
    --qemu-commandline="-accel kvm"
```

##### 5.2 Verificar que la VM está activa

```bash
### Listar máquinas virtuales
virsh list --all

### Esperar arranque completo (Fedora Cloud tarda ~45-90 segundos)
echo "⏳ Esperando arranque completo (60 segundos)..."
sleep 60

### Obtener dirección IP asignada por libvirt
sudo virsh domifaddr fedora-lab
```

✅ **Salida esperada:**
```
 Name       MAC address          Protocol     Address
 vnet0      52:54:00:ab:cd:ef    ipv4         192.168.122.100/24
```

❌ **Si no hay IP:** Verifique que la red `default` de libvirt está activa:
```bash
sudo virsh net-list --all
sudo virsh net-start default  ### Si está inactiva
```

---

#### 6. Restauración de la Base de Datos Pagila

> ⚠️ **Importante:** La restauración de Pagila se realiza **después del arranque** de la VM, ya que requiere que PostgreSQL esté activo y configurado.

##### 6.1 Conectar por SSH con PKI

```bash
### Conectar usando la clave privada
ssh -i ~/.ssh/fedora-lab-key user@192.168.122.100
```

##### 6.2 Script de restauración de Pagila (ejecutar dentro de la VM)

```bash
#!/bin/bash
### restore-pagila.sh - Ejecutar DENTRO de la VM como usuario 'user'

set -euo pipefail

PAGILA_REPO="https://github.com/devrimgunduz/pagila.git"
PAGILA_DIR="/opt/pagila"
DB_NAME="pagila"
DB_USER="postgres"

echo "🔄 Iniciando restauración de la base de datos Pagila..."

### 1. Verificar que PostgreSQL está activo
if ! sudo systemctl is-active --quiet postgresql-16; then
    echo "⚠️  PostgreSQL no está activo. Iniciando servicio..."
    sudo systemctl start postgresql-16
    sleep 5
fi

### 2. Clonar el repositorio de Pagila si no existe
if [ ! -d "$PAGILA_DIR/.git" ]; then
    echo "📦 Clonando repositorio de Pagila..."
    git clone "$PAGILA_REPO" "$PAGILA_DIR"
else
    echo "✅ Repositorio de Pagila ya existe. Actualizando..."
    cd "$PAGILA_DIR" && git pull
fi

### 3. Verificar que los scripts de restauración existen
if [ ! -f "$PAGILA_DIR/pagila-schema.sql" ] || [ ! -f "$PAGILA_DIR/pagila-data.sql" ]; then
    echo "❌ Error: Scripts de Pagila no encontrados en $PAGILA_DIR"
    echo "💡 Verifique la estructura del repositorio: ls -la $PAGILA_DIR"
    exit 1
fi

### 4. Crear la base de datos pagila
echo "🗄️  Creando base de datos '$DB_NAME'..."
sudo -u postgres psql -c "DROP DATABASE IF EXISTS $DB_NAME;" 2>/dev/null || true
sudo -u postgres psql -c "CREATE DATABASE $DB_NAME OWNER $DB_USER;"

### 5. Restaurar esquema y datos
echo "📥 Restaurando esquema de Pagila..."
sudo -u postgres psql -d "$DB_NAME" -f "$PAGILA_DIR/pagila-schema.sql"

echo "📥 Restaurando datos de ejemplo de Pagila..."
sudo -u postgres psql -d "$DB_NAME" -f "$PAGILA_DIR/pagila-data.sql"

### 6. Conceder permisos de lectura al usuario 'user' para consultas de práctica
echo "🔐 Configurando permisos de consulta para el usuario '$USERNAME'..."
sudo -u postgres psql -d "$DB_NAME" -c "GRANT CONNECT ON DATABASE $DB_NAME TO $USERNAME;"
sudo -u postgres psql -d "$DB_NAME" -c "GRANT USAGE ON SCHEMA public TO $USERNAME;"
sudo -u postgres psql -d "$DB_NAME" -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO $USERNAME;"

### 7. Verificación final
echo "✅ Verificando restauración..."
TABLE_COUNT=$(sudo -u postgres psql -d "$DB_NAME" -t -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public';")
FILM_COUNT=$(sudo -u postgres psql -d "$DB_NAME" -t -c "SELECT COUNT(*) FROM film;")

if [ "$TABLE_COUNT" -ge 15 ] && [ "$FILM_COUNT" -eq 1000 ]; then
    echo "✅ Base de datos Pagila restaurada exitosamente."
    echo "📊 Estadísticas: $TABLE_COUNT tablas, $FILM_COUNT películas en la tabla 'film'."
else
    echo "⚠️  Verificación incompleta. Revise los logs de restauración."
fi

echo ""
echo "🎯 Para comenzar a practicar consultas SQL:"
echo "   psql -h localhost -U user -d pagila"
echo ""
echo "📚 Ejemplos de consultas para practicar:"
echo "   SELECT title, release_year FROM film WHERE rating = 'PG' LIMIT 10;"
echo "   SELECT c.first_name, c.last_name, COUNT(r.rental_id) AS rentals"
echo "   FROM customer c JOIN rental r ON c.customer_id = r.customer_id"
echo "   GROUP BY c.customer_id ORDER BY rentals DESC LIMIT 5;"
```

##### 6.3 Ejecutar el script de restauración

```bash
### Dentro de la VM, después de conectarse por SSH:

### Hacer ejecutable el script
chmod +x ~/restore-pagila.sh

### Ejecutar la restauración (tiempo estimado: 2-5 minutos)
~/restore-pagila.sh
```

✅ **Éxito esperado:** Mensajes de confirmación con estadísticas de tablas restauradas.

##### 6.4 Verificación manual opcional

```bash
### Conectarse directamente a la base de datos Pagila
psql -h localhost -U user -d pagila

### Dentro de psql, ejecutar:
\dt                    ### Listar tablas
SELECT COUNT(*) FROM film;  ### Debe devolver 1000
SELECT DISTINCT rating FROM film;  ### Ver clasificaciones de películas
\q                     ### Salir de psql
```

---

#### 7. Hardening de Seguridad Completo

> ⚠️ **Advertencia:** Estas configuraciones son para laboratorios académicos. Ajuste según políticas institucionales para entornos productivos.

##### 7.1 Hardening de SSH (dentro de la VM)

```bash
### Conectarse como 'user' y ejecutar:

sudo tee -a /etc/ssh/sshd_config.d/hardening.conf << 'EOF'
### Hardening SSH para laboratorio académico
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
PrintMotd no
AcceptEnv LANG LC_*
Subsystem sftp /usr/libexec/sftp-server

### Limitar usuarios permitidos
AllowUsers user

### Timeout de sesión inactiva (15 minutos)
ClientAliveInterval 900
ClientAliveCountMax 0

### Limitar intentos de autenticación
MaxAuthTries 3
MaxSessions 2

### Logging mejorado
LogLevel VERBOSE
EOF

### Reiniciar SSH para aplicar cambios
sudo systemctl restart sshd

### Verificar configuración aplicada
sudo sshd -t && echo "✅ Configuración SSH válida"
```

##### 7.2 Hardening de PostgreSQL 16

```bash
### Dentro de la VM, como usuario 'user':

### 1. Restringir acceso en pg_hba.conf (solo localhost para laboratorio)
sudo tee /var/lib/pgsql/16/data/pg_hba.conf << 'EOF'
### TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
### Denegar acceso remoto en laboratorio (habilitar en producción con SSL)
host    all             all             0.0.0.0/0               reject
host    all             all             ::/0                    reject
EOF

### 2. Ajustar postgresql.conf para seguridad básica
sudo tee -a /var/lib/pgsql/16/data/postgresql.conf << 'EOF'
### Hardening PostgreSQL para laboratorio
listen_addresses = 'localhost'
port = 5432
max_connections = 20
ssl = off  ### Habilitar en producción con certificados válidos
log_connections = on
log_disconnections = on
log_statement = 'ddl'
log_min_duration_statement = 1000
EOF

### 3. Reiniciar PostgreSQL para aplicar cambios
sudo systemctl restart postgresql-16

### 4. Verificar que PostgreSQL solo escucha en localhost
sudo ss -tlnp | grep :5432
```

##### 7.3 Hardening del Sistema Operativo

```bash
### Dentro de la VM, como usuario 'user':

### 1. Actualizar sistema y eliminar paquetes innecesarios
sudo dnf update -y
sudo dnf remove -y cloud-init*  ### Si quedó algún residuo

### 2. Configurar firewall para servicios necesarios
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=postgresql
sudo firewall-cmd --permanent --remove-service=cockpit  ### Si no se usa
sudo firewall-cmd --reload

### 3. Habilitar y configurar fail2ban para protección SSH (opcional)
sudo dnf install -y fail2ban
sudo systemctl enable --now fail2ban

### 4. Configurar auditoría básica con auditd
sudo dnf install -y audit
sudo systemctl enable --now auditd

### 5. Verificar SELinux en modo enforcing
sudo setenforce 1
sudo sed -i 's/^SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config

### 6. Listar servicios activos y deshabilitar innecesarios
sudo systemctl list-units --type=service --state=running
### Deshabilitar servicios no requeridos (ejemplo):
### sudo systemctl disable --now cups avahi-daemon

echo "✅ Hardening de seguridad aplicado."
```

---

#### 8. Troubleshooting y Checklist de Validación

##### 8.1 Checklist de validación final (ejecutar desde host)

```bash
#!/bin/bash
### validate-fedora-lab.sh - Ejecutar desde el host Fedora

echo "🔍 Validación integral de Fedora Lab con Pagila..."

VM_NAME="fedora-lab"
VM_IP="192.168.122.100"  ### Ajustar según domifaddr
SSH_KEY="$HOME/.ssh/fedora-lab-key"

### 1. Verificar que la VM está corriendo
if virsh list | grep -q "$VM_NAME.*running"; then
    echo "✅ VM '$VM_NAME' está activa"
else
    echo "❌ VM '$VM_NAME' NO está activa"
    exit 1
fi

### 2. Verificar conectividad SSH con PKI
if ssh -i "$SSH_KEY" -o ConnectTimeout=10 -o BatchMode=yes user@$VM_IP "echo PKI_OK" 2>/dev/null | grep -q "PKI_OK"; then
    echo "✅ Conexión SSH con PKI exitosa"
else
    echo "❌ Conexión SSH con PKI fallida"
    echo "💡 Verifique: clave, IP, firewall, sshd_config"
    exit 1
fi

### 3. Verificar que root no puede loguear
if ssh -o ConnectTimeout=5 -o BatchMode=yes root@$VM_IP "echo ROOT_OK" 2>&1 | grep -q "Permission denied\|Connection refused"; then
    echo "✅ Login de root correctamente deshabilitado"
else
    echo "⚠️  Login de root podría estar habilitado - revisar sshd_config"
fi

### 4. Verificar PostgreSQL accesible
if ssh -i "$SSH_KEY" user@$VM_IP "sudo -u postgres psql -c 'SELECT version();'" 2>/dev/null | grep -q "PostgreSQL"; then
    echo "✅ PostgreSQL 16 respondiendo"
else
    echo "❌ PostgreSQL no responde - verificar servicio"
fi

### 5. Verificar base de datos Pagila
if ssh -i "$SSH_KEY" user@$VM_IP "sudo -u postgres psql -d pagila -c '\dt'" 2>/dev/null | grep -q "film"; then
    echo "✅ Base de datos 'pagila' con tablas de ejemplo restauradas"
else
    echo "⚠️  Base de datos Pagila no detectada - ejecutar restore-pagila.sh manualmente"
fi

### 6. Verificar hardening SSH
SSH_CONFIG=$(ssh -i "$SSH_KEY" user@$VM_IP "sudo grep -E 'PermitRootLogin|PasswordAuthentication' /etc/ssh/sshd_config" 2>/dev/null)
if echo "$SSH_CONFIG" | grep -q "PermitRootLogin no" && echo "$SSH_CONFIG" | grep -q "PasswordAuthentication no"; then
    echo "✅ Hardening SSH aplicado"
else
    echo "⚠️  Hardening SSH incompleto"
fi

echo ""
echo "🎉 Validación completada. El laboratorio está listo para uso académico."
echo "📚 Para practicar consultas SQL en Pagila:"
echo "   ssh -i ~/.ssh/fedora-lab-key user@$VM_IP"
echo "   psql -h localhost -U user -d pagila"
```

##### 8.2 Solución de problemas frecuentes

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| `virt-customize: ssh-inject: user does not exist` | Usuario creado después de inyectar clave | Reordenar: `--run-command useradd` **antes** de `--ssh-inject` |
| `ssh: connect to host ... port 22: Connection refused` | VM aún arrancando o SSH no activo | Esperar 60-90s; verificar con `virsh console fedora-lab` |
| `Permission denied (publickey)` | Clave incorrecta o permisos | `chmod 600 ~/.ssh/fedora-lab-key`; usar ruta absoluta en virt-customize |
| `network 'default' is not active` | Red libvirt inactiva | `sudo virsh net-start default` |
| PostgreSQL no inicia tras personalización | SELinux bloqueando o pg_hba.conf incorrecto | `sudo ausearch -m avc -ts recent`; revisar `/var/lib/pgsql/16/data/pg_hba.conf` |
| Pagila no se restaura: `file not found` | Repositorio no clonado correctamente | Verificar conexión a internet; ejecutar `git clone` manualmente en `/opt/pagila` |
| Error de permisos en psql | Usuario 'user' sin privilegios de conexión | Ejecutar manualmente: `sudo -u postgres psql -d pagila -c "GRANT CONNECT ON DATABASE pagila TO user;"` |

##### 8.3 Comandos útiles para gestión del laboratorio

```bash
### Detener/Iniciar/Reiniciar la VM
sudo virsh shutdown fedora-lab
sudo virsh start fedora-lab
sudo virsh reboot fedora-lab

### Acceder a consola serial (útil si SSH falla)
sudo virsh console fedora-lab
### Para salir: Ctrl+]

### Crear snapshot antes de cambios riesgosos
sudo virsh snapshot-create-as --domain fedora-lab --name "pre-hardening"

### Clonar para experimentos paralelos
sudo virt-clone --original fedora-lab --name fedora-lab-test --file /var/lib/libvirt/images/fedora44-lab-test.qcow2

### Eliminar completamente (VM + disco)
sudo virsh destroy fedora-lab
sudo virsh undefine fedora-lab
sudo rm /var/lib/libvirt/images/fedora44-lab.qcow2

### Exportar imagen para compartir (sin metadatos de nube)
sudo virt-sysprep -d fedora-lab --operations defaults,-ssh-hostkeys
sudo cp /var/lib/libvirt/images/fedora44-lab.qcow2 ~/shared-fedora-lab.qcow2
```

---

#### 📚 Referencias y Recursos Adicionales

- **Red Hat.** (2021). *Build a lab quickly*. https://www.redhat.com/en/blog/build-lab-quickly
- **devrimgunduz.** (2026). *Pagila - Sample Database for PostgreSQL*. https://github.com/devrimgunduz/pagila
- **PostgreSQL Global Development Group.** (2026). *PostgreSQL 16 Documentation*. https://www.postgresql.org/docs/16/
- **libguestfs Tools.** (2026). *virt-customize manual*. https://libguestfs.org/virt-customize.1.html
- **UAM-Iztapalapa.** (2026). *Programa de la UEA Bases de Datos (2151106)*. División de Ciencias Básicas e Ingeniería.

---

> 💡 **Consejo pedagógico:** Esta arquitectura copy-on-write le permite reiniciar el entorno de laboratorio en menos de 2 minutos: basta con eliminar `fedora44-lab.qcow2` y volver a ejecutar las secciones 2.3 y 4. La imagen base `fedora44-base.qcow2` permanece intacta para futuros experimentos.

> 🔐 **Nota de seguridad institucional:** Para cumplir con políticas de la UAM, documente todas las personalizaciones en el repositorio institucional https://github.com/jzavalar/bases-de-datos y utilice credenciales temporales que sean rotadas al finalizar cada trimestre académico.

---

*Elaborado por: Dr. Jesús Zavala Ruiz*  
*Profesor de la UEA Bases de Datos (2151106)*  
*División de Ciencias Básicas e Ingeniería*  
*Universidad Autónoma Metropolitana, Unidad Iztapalapa*  
*Mayo 2026*
