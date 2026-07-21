### Laboratorio 06. Instalación de pgAdmin y DBeaver en Fedora 44 para Conectarse a la Base de Datos 

**Unidad de Enseñanza-Aprendizaje:** Bases de Datos (2151106)  
**Institución:** Universidad Autónoma Metropolitana, Unidad Iztapalapa  
**Autor:** dr. Jesús Zavala Ruiz  
**Creación:** 27 de mayo 2026
**Última actualización:** 21 de julio 2026  

**Entorno de ejecución:** Fedora 44 Cloud Base sobre KVM/libvirt  
**Dirección de la máquina virtual:** 192.168.122.24  
**Usuario de acceso:** alumno
**Acceso:** por túnel SSH con llave privada y passfrase nula

---

#### Introducción

La administración de sistemas gestores de bases de datos en contextos académicos y profesionales demanda herramientas que permitan interactuar con el motor de forma intuitiva, visualizar estructuras de datos complejas y ejecutar consultas sin depender exclusivamente de interfaces de línea de comandos. pgAdmin 4 constituye la plataforma de administración gráfica oficial para PostgreSQL, diseñada bajo una arquitectura web que facilita la gestión simultánea de múltiples instancias de bases de datos desde un navegador estándar.

El presente laboratorio documenta el procedimiento completo para desplegar pgAdmin 4 en modalidad web sobre Fedora 44. El proceso integra la instalación de componentes base, la aplicación de controles de seguridad mediante SELinux, la configuración de acceso restringido a través del servidor web Apache y el establecimiento de una capa de filtrado de red con firewalld. La arquitectura resultante replica prácticas de endurecimiento utilizadas en entornos productivos, garantizando que la interfaz de administración permanezca accesible mientras el motor de bases de datos se mantiene aislado y protegido.

#### 0. Procedimiento de desinstalación (para entornos de prueba)

En contextos de laboratorio y desarrollo, es frecuente requerir la reinstalación limpia de componentes para validar procedimientos, documentar pasos o recuperar un estado conocido tras pruebas fallidas. Esta sección describe el procedimiento sistemático para eliminar completamente pgAdmin 4 y sus componentes asociados, preservando el resto del entorno operativo (PostgreSQL, Apache base, configuración de red).

#### 0.1. Detención de servicios relacionados

Antes de remover paquetes, detenga los servicios que podrían estar utilizando archivos de pgAdmin para evitar conflictos durante la desinstalación:

```bash
### Detener Apache temporalmente para liberar archivos en uso
sudo systemctl stop httpd
```

Salida:
```
pgadmin4-9.15-1.fc44.x86_64
pgadmin4-httpd-9.15-1.fc44.x86_64
```

Verifique que no queden procesos de pgAdmin activos en memoria:

```bash
### Verificar que no queden procesos de pgAdmin activos
pgrep -f pgadmin || echo "No hay procesos de pgAdmin activos"
```

Salida:
```
No hay procesos de pgAdmin activos
```

#### 0.2. Eliminación de paquetes de pgAdmin

Remueva los paquetes específicos de pgAdmin utilizando el gestor de paquetes `dnf`. Este proceso eliminará los binarios, bibliotecas Python y archivos de configuración gestionados por el sistema de paquetes:

```bash
### Listar paquetes instalados relacionados con pgAdmin para confirmación
rpm -qa | grep pgadmin4
```

Salida:
```
pgadmin4-9.15-1.fc44.x86_64
pgadmin4-httpd-9.15-1.fc44.x86_64
```

Ejecute la remoción de paquetes manteniendo las dependencias compartidas que podrían ser utilizadas por otros servicios:

```bash
### Eliminar paquetes de pgAdmin manteniendo dependencias compartidas
sudo dnf remove -y pgadmin4 pgadmin4-httpd
```

Salida:
```
Package                           Arch   Version                  Reposito      Size
Removing:
 pgadmin4                         x86_64 0:9.15-1.fc44            updates   45.3 MiB
 pgadmin4-httpd                   x86_64 0:9.15-1.fc44            updates  311.0   B
Removing unused dependencies:
 fribidi                          x86_64 0:1.0.16-4.fc44          fedora   190.0 KiB
 jbigkit-libs                     x86_64 0:2.1-33.fc44            fedora   117.2 KiB
 lcms2                            x86_64 0:2.16-7.fc44            fedora   445.7 KiB
 ...
 libxcb                           x86_64 0:1.17.0-7.fc44          fedora     1.1 MiB
 openjpeg                         x86_64 0:2.5.4-3.fc44           fedora   464.2 KiB
 python3-alembic                  noarch 0:1.18.4-1.fc44          fedora     3.0 MiB
 ...
 python3-wsproto                  noarch 0:1.2.0-15.fc44          fedora   271.8 KiB
 python3-wtforms                  noarch 0:3.0.1-20.fc44          fedora   714.9 KiB

Transaction Summary:
 Removing:         142 packages

After this operation, 492 MiB will be freed (install 0 B, remove 492 MiB).
Running transaction
[  1/143] Prepare transaction               100% | 710.0   B/s | 142.0   B |  00m00s
...
[  5/143] Removing pgadmin4-httpd-0:9.15-1. 100% | 272.0   B/s |   3.0   B |  00m00s
>>> [RPM] /etc/httpd/conf.d/pgadmin4.conf saved as /etc/httpd/conf.d/pgadmin4.conf.rpmsave
[  6/143] Removing pgadmin4-0:9.15-1.fc44.x 100% |  28.0 KiB/s |   6.5 KiB |  00m00s
...
[141/143] Removing python3-markupsafe-0:3.0 100% |   2.9 KiB/s |  24.0   B |  00m00s
[142/143] Removing python3-brotli-0:1.2.0-3 100% | 933.0   B/s |  14.0   B |  00m00s
[143/143] Removing python3-mod_wsgi-0:5.0.2 100% |   1.3 KiB/s | 435.0   B |  00m00s
Complete!
[alumno@fedora-lab ~]$ 
```

Confirme que los paquetes fueron removidos correctamente:

```bash
### Verificar que los paquetes fueron removidos
rpm -q pgadmin4 pgadmin4-httpd 2>&1 || echo "Paquetes de pgAdmin eliminados correctamente"
```

Salida:
```
package pgadmin4 is not installed
package pgadmin4-httpd is not installed
Paquetes de pgAdmin eliminados correctamente
```

> Nota técnica: El comando `dnf remove` preserva las dependencias que podrían ser utilizadas por otros paquetes (como `httpd`, `python3-flask`, etc.). No elimine `httpd` a menos que esté seguro de que no es requerido por otros servicios en el entorno.

#### 0.3. Eliminación de archivos de configuración y datos residuales

Los paquetes RPM no eliminan automáticamente los directorios de datos y registros generados durante la ejecución de la aplicación. Remuélvalos manualmente para garantizar un estado limpio:

```bash
### Eliminar directorios de datos y registros de pgAdmin
sudo rm -rf /var/lib/pgadmin
sudo rm -rf /var/log/pgadmin

### Eliminar archivos de configuración de Apache específicos de pgAdmin
sudo rm -f /etc/httpd/conf.d/pgadmin4.conf
sudo rm -f /etc/httpd/conf.d/pgadmin4.conf.bak

### Verificar que los archivos fueron eliminados
ls -la /var/lib/pgadmin /var/log/pgadmin /etc/httpd/conf.d/pgadmin4.conf 2>&1 || echo "Archivos residuales eliminados"
```

Salida:
```
ls: cannot access '/var/lib/pgadmin': No such file or directory
ls: cannot access '/var/log/pgadmin': No such file or directory
ls: cannot access '/etc/httpd/conf.d/pgadmin4.conf': No such file or directory
Archivos residuales eliminados
```

#### 0.4. Limpieza de definiciones de contexto SELinux

Si aplicó políticas personalizadas de SELinux durante una instalación previa, elimine las reglas definidas para evitar conflictos en futuras instalaciones:

```bash
### Listar reglas personalizadas relacionadas con pgAdmin
sudo semanage fcontext -l | grep pgadmin
```

Salida:
```
/var/lib/pgadmin(/.*)?                             all files          system_u:object_r:httpd_sys_rw_content_t:s0 
/var/log/pgadmin(/.*)?                             all files          system_u:object_r:httpd_log_t:s0 
```

Elimine las definiciones de contexto para los directorios de pgAdmin:

```bash
### Eliminar definiciones de contexto para directorios de pgAdmin
sudo semanage fcontext -d "/var/log/pgadmin(/.*)?"
sudo semanage fcontext -d "/var/lib/pgadmin(/.*)?"

### Verificar que las reglas fueron removidas
sudo semanage fcontext -l | grep pgadmin || echo "Reglas SELinux de pgAdmin eliminadas"
```

Salida:
```
Reglas SELinux de pgAdmin eliminadas
```

> Nota técnica: El comando `semanage fcontext -d` elimina la definición de la política, pero no modifica los archivos existentes. Dado que los directorios fueron eliminados en el paso anterior, no es necesario ejecutar `restorecon`.

#### 0.5. Reversión de reglas de firewalld (si aplica)

Si configuró `firewalld` específicamente para pgAdmin, elimine las reglas asociadas para restaurar la configuración original:

```bash
### Verificar si el servicio HTTP está habilitado
sudo firewall-cmd --list-services | grep http
```

Salida:
```
dhcpv6-client http mdns ssh
```

Remueva el servicio HTTP si fue agregado exclusivamente para pgAdmin:

```bash
### Advertencia: Si Apache sirve otras aplicaciones web, no ejecute este comando
sudo firewall-cmd --permanent --remove-service=http
sudo firewall-cmd --reload

### Verificar que la regla fue eliminada
sudo firewall-cmd --list-services
```

> Precaución: Solo elimine la regla `http` si Apache no sirve otras aplicaciones web. Si `httpd` se utiliza para otros propósitos, mantenga la regla y documente esta excepción en la bitácora de laboratorio.

#### 0.6. Limpieza de booleanes SELinux modificados (opcional)

Si habilitó booleanes específicamente para pgAdmin y desea revertirlos a sus valores por defecto:

```bash
### Verificar estado actual del booleane
getsebool httpd_can_network_connect
```

Salida:
```
httpd_can_network_connect --> on
```

Revierta el booleane a su valor por defecto si no es requerido por otros servicios:

```bash
### Revertir a valor por defecto (off) si no es requerido por otros servicios
sudo setsebool -P httpd_can_network_connect off

### Verificar cambio
getsebool httpd_can_network_connect
```

> Nota técnica: El booleane `httpd_can_network_connect` puede ser requerido por otras aplicaciones web que necesiten realizar conexiones salientes. Evalúe si otros servicios en su entorno dependen de esta configuración antes de revertirla.

#### 0.7. Reinicio de Apache para aplicar limpieza

Reinicie el servidor web para que cargue la configuración actualizada sin las directivas de pgAdmin:

```bash
sudo systemctl start httpd
sudo systemctl is-active httpd
```

Salida:
```
active
```

#### 0.8. Verificación del estado limpio

Ejecute las siguientes comprobaciones para confirmar que pgAdmin fue completamente removido del sistema:

```bash
### Verificar que no existen paquetes de pgAdmin
rpm -qa | grep pgadmin4 || echo "✓ Sin paquetes de pgAdmin"
```

Salida:
```
✓ Sin paquetes de pgAdmin
```

```bash
### Verificar que no existen directorios de datos
ls -d /var/lib/pgadmin /var/log/pgadmin 2>&1 || echo "✓ Sin directorios de pgAdmin"
```

Salida:
```
ls: cannot access '/var/lib/pgadmin': No such file or directory
ls: cannot access '/var/log/pgadmin': No such file or directory
✓ Sin directorios de pgAdmin
```

```bash
### Verificar que no existe configuración de Apache para pgAdmin
ls /etc/httpd/conf.d/pgadmin4.conf 2>&1 || echo "✓ Sin configuración Apache para pgAdmin"
```

Salida:
```
✓ Sin configuración Apache para pgAdmin
```

```bash
### Verificar que Apache responde sin errores de configuración
sudo httpd -t
```

Salida esperada para `httpd -t`:
```
Syntax OK
```

#### 0.9. Comandos de verificación rápida (resumen)

Ejecute este bloque único para una validación rápida del estado limpio:

```bash
echo "=== Verificación de desinstalación de pgAdmin ==="
rpm -qa | grep pgadmin4 || echo "✓ Paquetes: eliminados"
ls -d /var/lib/pgadmin /var/log/pgadmin 2>&1 | grep "No such file" && echo "✓ Directorios: eliminados"
ls /etc/httpd/conf.d/pgadmin4.conf 2>&1 | grep "No such file" && echo "✓ Configuración Apache: eliminada"
sudo httpd -t 2>&1 | grep "Syntax OK" && echo "✓ Apache: configuración válida"
echo "=== Estado limpio confirmado ==="
```

Salida:
```
=== Verificación de desinstalación de pgAdmin ===
✓ Paquetes: eliminados
ls: cannot access '/var/lib/pgadmin': No such file or directory
ls: cannot access '/var/log/pgadmin': No such file or directory
✓ Directorios: eliminados
ls: cannot access '/etc/httpd/conf.d/pgadmin4.conf': No such file or directory
✓ Configuración Apache: eliminada
Syntax OK
✓ Apache: configuración válida
=== Estado limpio confirmado ===
```

#### 0.10. Consideraciones finales del procedimiento de desinstalación

- **PostgreSQL permanece intacto**: Este procedimiento no afecta la instalación de PostgreSQL, la base de datos `pagila` ni los usuarios configurados.
- **Apache base se preserva**: El servidor web `httpd` continúa operativo para otros servicios si así lo requiere.
- **Claves SSH no se modifican**: Las credenciales de acceso remoto (`~/.ssh/fedora-lab-key`) permanecen válidas.
- **Configuración de red se mantiene**: Las reglas de libvirt y la dirección IP `192.168.122.24` no son alteradas.

Con el entorno en estado limpio, puede proceder a documentar el procedimiento de instalación desde cero siguiendo las secciones subsiguientes de este laboratorio. Se recomienda capturar la salida de cada comando durante la reinstalación para validar que el proceso es reproducible y libre de dependencias residuales.

---

#### 1. Requisitos del entorno

Antes de iniciar el procedimiento de instalación, verifique que la máquina virtual cumple con las siguientes condiciones operativas:

- La VM `fedora44-lab` ejecuta Fedora 44 Cloud Base actualizado.
- El usuario `alumno` posee privilegios de administración mediante sudo.
- PostgreSQL 18 se encuentra operativo con la base de datos de ejemplo `pagila` cargada.
- Existe una clave SSH configurada para acceso remoto desde el equipo anfitrión.
- La conexión de red está gestionada por libvirt en modalidad NAT o puente.

#### 2. Instalación de componentes base en la máquina virtual

#### 2.1. Inicio y conexión a la VM

Inicie la máquina virtual desde el sistema anfitrión:

```bash
sudo virsh start fedora44-lab
```

Salida:
```
[sudo] password for alumno: 
Domain 'fedora44-lab' started
```

Establezca una sesión SSH hacia la máquina virtual utilizando la clave configurada:

```bash
### Conectarse a la VM
ssh -i ~/.ssh/fedora-lab-key alumno@192.168.122.24
```

#### 2.2. Preparación del repositorio y verificación de paquetes

pgAdmin requiere un repositorio específico para su instalación en Fedora. Agregue el repositorio oficial:

```bash
sudo rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-fedora-repo-2-1.noarch.rpm
```

Verifique el estado inicial de los paquetes requeridos antes de la instalación:

```bash
rpm -q pgadmin4 pgadmin4-httpd httpd
```

Salida:
```
package pgadmin4 is not installed
package pgadmin4-httpd is not installed
httpd-2.4.67-1.fc44.x86_64
```

**Nota técnica:** El paquete `pgadmin4-httpd` es mantenido por la comunidad de Fedora y está diseñado específicamente para integrar pgAdmin 4 con el servidor web Apache. Incluye la configuración WSGI (`/etc/httpd/conf.d/pgadmin4.conf`) adaptada a las rutas y políticas de Fedora, declara dependencias explícitas con `httpd`, `python3-mod_wsgi` y los contextos SELinux correspondientes. Su instalación coloca automáticamente el archivo de configuración necesario para que Apache sirva pgAdmin en la ruta `/pgadmin`.

#### 2.3. Instalación de pgAdmin y servidor web

Actualice los metadatos del sistema e instale los componentes necesarios:

```bash
sudo dnf update -y
sudo dnf install -y pgadmin4 pgadmin4-httpd httpd
```

Salida:
```
Updating and loading repositories:
 Fedora 44 - x86_64 - Updates               100% |   4.0 KiB/s |   6.7 KiB |  00m02s
Repositories loaded.
Package "httpd-2.4.67-1.fc44.x86_64" is already installed.

Package                           Arch   Version                  Reposito      Size
Installing:
 pgadmin4                         x86_64 0:9.15-1.fc44            updates   45.3 MiB
 pgadmin4-httpd                   x86_64 0:9.15-1.fc44            updates  311.0   B
Installing dependencies:
 fribidi                          x86_64 0:1.0.16-4.fc44          fedora   190.0 KiB
 jbigkit-libs                     x86_64 0:2.1-33.fc44            fedora   117.2 KiB
 lcms2                            x86_64 0:2.16-7.fc44            fedora   445.7 KiB
...
 python3-qrcode+all               noarch 0:8.0-10.fc44            fedora     0.0   B
 python3-urllib3+socks            noarch 0:2.7.0-1.fc44           updates    0.0   B

Transaction Summary:
 Installing:       142 packages

Total size of inbound packages is 67 MiB. Need to download 67 MiB.
After this operation, 492 MiB extra will be used (install 492 MiB, remove 0 B).
[  1/142] python3-azure-identity-1:1.17.1-7 100% | 167.9 KiB/s | 267.9 KiB |  00m02s
[  2/142] python3-authlib-0:1.5.2-2.fc44.no 100% | 308.1 KiB/s | 507.4 KiB |  00m02s
...
[141/142] libtommath-0:1.3.1~rc1-7.fc44.x86 100% | 495.0 KiB/s |  64.8 KiB |  00m00s
[142/142] python3-keyring+completion-0:25.7 100% |   3.9 KiB/s |  14.3 KiB |  00m04s
------------------------------------------------------------------------------------
[142/142] Total                             100% |   3.1 MiB/s |  67.3 MiB |  00m22s
Running transaction
[  1/144] Verify package files              100% | 396.0   B/s | 142.0   B |  00m00s
[  2/144] Prepare transaction               100% | 473.0   B/s | 142.0   B |  00m00s
...
[143/144] Installing python3-urllib3+socks- 100% |  13.5 KiB/s | 124.0   B |  00m00s
[144/144] Installing python3-crypto-0:2.6.1 100% |   3.6 MiB/s |   2.3 MiB |  00m01s
Complete!
```

Confirme que los paquetes se instalaron correctamente:

```bash
rpm -q pgadmin4 pgadmin4-httpd httpd
```

Salida esperada:
```
pgadmin4-9.15-1.fc43.x86_64
pgadmin4-httpd-9.15-1.fc43.x86_64
httpd-2.4.67-1.fc43.x86_64
```

#### 3. Activación y verificación del servidor web Apache

Una vez instalados los paquetes, habilite el servicio de Apache para que inicie automáticamente con el sistema y actívelo de forma inmediata:

```bash
sudo systemctl enable --now httpd
```

> Nota: Una salida sin mensajes de error indica que la operación se completó correctamente.

Verifique el estado operativo del servicio:

```bash
sudo systemctl is-active httpd
```

Salida:
```
active
```

Confirme que el servidor web está escuchando en el puerto estándar:

```bash
sudo ss -tlnp | grep :80
```

Salida:
```
LISTEN 0      511                *:80              *:*    users:(("httpd",pid=1390,fd=4),("httpd",pid=1029,fd=4),("httpd",pid=1027,fd=4),("httpd",pid=1026,fd=4),("httpd",pid=948,fd=4))
```

La respuesta `active` y la línea `LISTEN ... *:80` confirman que Apache está operando y recibiendo conexiones en el puerto HTTP estándar.

#### 4. Configuración de acceso a pgAdmin en Apache

#### 4.1. Modificación de la directiva de control de acceso

Por defecto, la configuración de pgAdmin en Fedora restringe el acceso a la interfaz web exclusivamente desde la dirección local (`localhost`). Para permitir la conexión desde el equipo anfitrión a través de la red virtual, es necesario ampliar la directiva de control de acceso en la configuración de Apache.

Realice un respaldo del archivo original antes de realizar modificaciones:

```bash
sudo cp /etc/httpd/conf.d/pgadmin4.conf /etc/httpd/conf.d/pgadmin4.conf.bak
sudo sed -i 's/Require local/Require ip 192.168.122.0\/24 127.0.0.1 ::1/' /etc/httpd/conf.d/pgadmin4.conf
grep "Require" /etc/httpd/conf.d/pgadmin4.conf
```

Salida:
```
        Require ip 192.168.122.0/24 127.0.0.1 ::1
```

Esta modificación autoriza explícitamente las solicitudes provenientes de la subred `192.168.122.0/24`, manteniendo simultáneamente el acceso local para fines de diagnóstico. Es importante notar que la ruta de acceso configurada es `/pgadmin`, por lo que la URL final de acceso será `http://192.168.122.24/pgadmin/`. En este punto, la URL aún no está accesible porque falta inicializar la aplicación pgAdmin.

#### 5. Inicialización de pgAdmin en la máquina virtual

#### 5.1. Localización y uso del script de configuración

pgAdmin requiere una base de datos interna en formato SQLite para almacenar credenciales de usuarios, configuraciones de servidores registrados y preferencias de sesión. Fedora distribuye un script de administración basado en Python que gestiona esta inicialización.

Localice la ruta del script de configuración:

```bash
rpm -ql pgadmin4 | grep '/setup\.py$' | head -n 1
```

Salida:
```
/usr/lib/pgadmin4/setup.py
```

Consulte las opciones disponibles en el script:

```bash
sudo python3 /usr/lib/pgadmin4/setup.py --help
```

Salida:
```                                                                                    
 Usage: setup.py [OPTIONS] COMMAND [ARGS]...                                        
                                                                                    
╭─ Options ────────────────────────────────────────────────────────────────────────╮
│ --install-completion          Install completion for the current shell.          │
│ --show-completion             Show completion for the current shell, to copy it  │
│                               or customize the installation.                     │
│ --help                        Show this message and exit.                        │
╰──────────────────────────────────────────────────────────────────────────────────╯
╭─ Commands ───────────────────────────────────────────────────────────────────────╮
│ dump-servers           Dump the server groups and servers.                       │
│ load-servers           Load server groups and servers.                           │
│ load-users             Load users from a JSON file.                              │
│ add-user               Add Internal user.                                        │
│ add-external-user      Add external user, other than Internal like               │
│                        Ldap, Ouath2, Kerberos, Webserver.                        │
│ delete-user            Delete the user.                                          │
│ update-user            Update internal user.                                     │
│ get-users                                                                        │
│ update-external-user   Update external users other than Internal like            │
│                        Ldap, Ouath2, Kerberos, Webserver.                        │
│ get-prefs                                                                        │
│ set-prefs              Set User preferences.                                     │
│ setup-db               Setup the configuration database.                         │
│ cleanup-session-files  Delete expired session files.                             │
╰──────────────────────────────────────────────────────────────────────────────────╯
```

#### 5.2. Creación de la base de datos interna

Entre las opciones listadas, el subcomando `setup-db` es el responsable de crear la estructura de almacenamiento interno. Ejecute la inicialización:

```bash
sudo python3 /usr/lib/pgadmin4/setup.py setup-db
```

Salida:
```
NOTE: Configuring authentication for SERVER mode.

Enter the email address and password to use for the initial pgAdmin user account:

Email address: jzr@xanum.uam.mx
Password: 
Retype password:
pgAdmin 4 - Application Initialisation
======================================
```

El proceso es idempotente y no produce salida adicional si se completa correctamente. La ausencia de mensajes de error indica que la base de datos interna `pgadmin4.db` ha sido generada en `/var/lib/pgadmin/`.

#### 6. Ajuste de permisos y políticas SELinux

#### 6.1. Contexto de seguridad en Fedora

Fedora implementa Security-Enhanced Linux (SELinux) en modo obligatorio (`Enforcing`). Esta capa de seguridad evalúa no solo los permisos tradicionales del sistema de archivos, sino también los contextos de seguridad asociados a cada archivo y directorio. Si el proceso de Apache (`httpd_t`) intenta escribir en un directorio que posee un contexto genérico, la operación será bloqueada independientemente de los permisos `rwx` asignados.

#### 6.2. Instalación de herramientas de gestión de políticas

Instale las herramientas necesarias para gestionar políticas SELinux:

```bash
sudo dnf install -y policycoreutils-python-utils
```

Salida:
```
Updating and loading repositories:
Repositories loaded.
Package "policycoreutils-python-utils-3.10-1.fc44.noarch" is already installed.

Nothing to do.
```

#### 6.3. Configuración de permisos básicos

Ajuste la propiedad y los permisos de los directorios utilizados por pgAdmin:

```bash
sudo mkdir -p /var/lib/pgadmin /var/log/pgadmin
sudo chown -R apache:apache /var/lib/pgadmin /var/log/pgadmin
sudo chmod -R 750 /var/lib/pgadmin /var/log/pgadmin
```

#### 6.4. Aplicación de contextos SELinux específicos

Defina y aplique los contextos de seguridad requeridos para que Apache pueda operar sobre los archivos de pgAdmin:

```bash
sudo semanage fcontext -a -t httpd_log_t "/var/log/pgadmin(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/lib/pgadmin(/.*)?"
sudo restorecon -Rv /var/log/pgadmin /var/lib/pgadmin
```

Salida:
```
Relabeled /var/log/pgadmin from system_u:object_r:var_log_t:s0 to system_u:object_r:httpd_log_t:s0
Relabeled /var/log/pgadmin/pgadmin4.log from unconfined_u:object_r:var_log_t:s0 to unconfined_u:object_r:httpd_log_t:s0
Relabeled /var/lib/pgadmin from system_u:object_r:var_lib_t:s0 to system_u:object_r:httpd_sys_rw_content_t:s0
Relabeled /var/lib/pgadmin/sessions from unconfined_u:object_r:var_lib_t:s0 to unconfined_u:object_r:httpd_sys_rw_content_t:s0
Relabeled /var/lib/pgadmin/storage from unconfined_u:object_r:var_lib_t:s0 to unconfined_u:object_r:httpd_sys_rw_content_t:s0
Relabeled /var/lib/pgadmin/azurecredentialcache from unconfined_u:object_r:var_lib_t:s0 to unconfined_u:object_r:httpd_sys_rw_content_t:s0
Relabeled /var/lib/pgadmin/pgadmin4.db from unconfined_u:object_r:var_lib_t:s0 to unconfined_u:object_r:httpd_sys_rw_content_t:s0
```

#### 6.5. Verificación de contextos aplicados

Verifique que los contextos de seguridad se aplicaron correctamente:

```bash
sudo ls -laZ /var/log/pgadmin/
```

Salida:
```
total 0
drwxr-x---. 1 apache apache system_u:object_r:httpd_log_t:s0      24 May 27 04:30 .
drwxr-xr-x. 1 root   root   system_u:object_r:var_log_t:s0       306 May 27 04:06 ..
-rwxr-x---. 1 apache apache unconfined_u:object_r:httpd_log_t:s0   0 May 27 04:30 pgadmin4.log
```

```bash
sudo ls -laZ /var/lib/pgadmin/
```

Salida:
```
total 196
drwxr-x---. 1 apache apache system_u:object_r:httpd_sys_rw_content_t:s0         92 May 27 04:40 .
drwxr-xr-x. 1 root   root   system_u:object_r:var_lib_t:s0                     442 May 27 04:06 ..
drwxr-x---. 1 apache apache unconfined_u:object_r:httpd_sys_rw_content_t:s0      0 May 27 04:30 azurecredentialcache
-rwxr-x---. 1 apache apache unconfined_u:object_r:httpd_sys_rw_content_t:s0 200704 May 27 04:40 pgadmin4.db
drwxr-x---. 1 apache apache unconfined_u:object_r:httpd_sys_rw_content_t:s0      0 May 27 04:30 sessions
drwxr-x---. 1 apache apache unconfined_u:object_r:httpd_sys_rw_content_t:s0      0 May 27 04:30 storage
```

La salida debe mostrar `httpd_log_t` para los directorios de registro y `httpd_sys_rw_content_t` para los directorios de datos.

#### 6.6. Habilitación de booleanes SELinux

Active el booleane que permite a Apache establecer conexiones de red salientes, requisito para el funcionamiento completo de pgAdmin:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

#### 7. Configuración del cortafuegos (firewalld)

#### 7.1. Instalación y activación del servicio

Las imágenes Cloud Base de Fedora no incluyen el gestor de cortafuegos por defecto. Instálelo y configúrelo para autorizar únicamente el tráfico HTTP necesario:

```bash
sudo dnf install -y firewalld
```

Salida:
```
Updating and loading repositories:
Repositories loaded.
Package "firewalld-2.4.0-2.fc44.noarch" is already installed.

Nothing to do.
```

Habilite e inicie el servicio de cortafuegos:

```bash
sudo systemctl enable --now firewalld
```

#### 7.2. Verificación del estado y configuración de reglas

Verifique que el servicio está operando:

```bash
sudo systemctl is-active firewalld
```

Salida:
```
active
```

Revise la configuración de la zona activa:

```bash
sudo firewall-cmd --list-all
```

Salida:
```
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp1s0
  sources: 
  services: dhcpv6-client http mdns ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

#### 7.3. Habilitación explícita del servicio HTTP

Si el servicio `http` no aparece en la lista de servicios permitidos, agréguelo de forma permanente y recargue las reglas:

```bash
sudo firewall-cmd --permanent --add-service=http
```

Salida:
```
success
```

```bash
sudo firewall-cmd --reload
```

Salida:
```
success
```

Verifique que la regla se aplicó correctamente:

```bash
sudo firewall-cmd --list-services
```

Salida:
```
dhcpv6-client http mdns ssh
```

La presencia de `http` junto a servicios base como `ssh` confirma que el tráfico web está autorizado. Confirme que no existen puertos expuestos de forma numérica directa:

```bash
sudo firewall-cmd --list-ports
```

Una salida vacía o sin referencias al puerto `5432` confirma que el motor de bases de datos permanece protegido detrás del túnel SSH, cumpliendo con el principio de mínimo privilegio.

#### 8. Validación de acceso web

#### 8.1. Reinicio y verificación del servicio Apache

Reinicie el servidor web para consolidar todos los cambios de configuración aplicados:

```bash
sudo systemctl restart httpd
```

Verifique el estado operativo:

```bash
sudo systemctl is-active httpd
```

Salida:
```
active
```

Confirme que Apache escucha en el puerto 80:

```bash
sudo ss -tlnp | grep :80
```

Salida:
```
LISTEN 0      511                *:80              *:*    users:(("httpd",pid=5377,fd=4),("httpd",pid=5376,fd=4),("httpd",pid=5370,fd=4),("httpd",pid=5367,fd=4))
```

#### 8.2. Prueba de respuesta HTTP local

Verifique que la aplicación pgAdmin responde correctamente a solicitudes HTTP:

```bash
curl -s -o /dev/null -w "HTTP Code: %{http_code}\n" http://localhost/pgadmin/
```

Salida:
```
HTTP Code: 302
```

Obtenga detalles de la respuesta para confirmar la redirección al módulo de autenticación:

```bash
curl -v http://localhost/pgadmin/ 2>&1 | grep -E "HTTP/|Location:"
```

Salida:
```
* using HTTP/1.x
> GET /pgadmin/ HTTP/1.1
< HTTP/1.1 302 FOUND
< Location: /pgadmin/login?next=/pgadmin/
```

Un código de respuesta `302` con redirección hacia `/pgadmin/login` confirma que la pila está operativa y que el sistema solicita autenticación. Este comportamiento es esperado en la primera carga de la aplicación.

#### 8.3. Verificación de conectividad desde el equipo anfitrión

Desde el sistema anfitrión, confirme que la interfaz web es accesible a través de la red virtual:

```bash
curl -s -o /dev/null -w "HTTP Code: %{http_code}\n" http://192.168.122.24/pgadmin/
```

Salida:
```
HTTP Code: 302
```

Acceda mediante un navegador web a la dirección `http://192.168.122.24/pgadmin/`. Deberá visualizar la interfaz de autenticación de pgAdmin:

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/lab_06_pgadmin-desde-localhost.png" width="70%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 1. Acceso a pgAdmin desde el anfitrión</div>
</div>

#### 9. Conexión segura a PostgreSQL mediante túnel SSH

#### 9.1. Establecimiento del túnel SSH desde el anfitrión

Por diseño, PostgreSQL escucha únicamente en la interfaz local de la máquina virtual. Para administrar la base de datos desde el equipo anfitrión sin exponer el puerto `5432` en la red, establezca un túnel SSH cifrado:

```bash
ssh -L 5433:localhost:5432 -i ~/.ssh/fedora-lab-key alumno@192.168.122.24 -N -f
ss -tlnp | grep 5433
```

El parámetro `-L 5433:localhost:5432` redirige el puerto local `5433` del anfitrión hacia el puerto `5432` de la máquina virtual. La bandera `-f` ejecuta el proceso en segundo plano y `-N` inhibe la ejecución de comandos remotos. La presencia de `LISTEN ... 127.0.0.1:5433` confirma que el canal seguro está activo.

#### 9.2. Validación de conexión mediante cliente psql

Verifique la versión del cliente PostgreSQL instalado en el anfitrión:

```bash
psql --version
```

Salida:
```
psql (PostgreSQL) 16.13
```

Pruebe la conexión a la base de datos a través del túnel, verificando los parámetros de sesión:

```bash
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT current_user, current_database(), inet_server_port();"
```

Salida:
```
 current_user | current_database | inet_server_port 
--------------+------------------+------------------
 alumno       | pagila           |             5432
(1 row)
```

Consulte la versión del motor de bases de datos remoto:

```bash
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT version();"
```

Salida:
```

                                                   version                          
                          
--------------------------------------------------------------------------------------------------------------
 PostgreSQL 18.3 on x86_64-redhat-linux-gnu, compiled by gcc (GCC) 16.0.1 20260321 (Red Hat 16.0.1-0), 64-bit
(1 row)
```

Valide la integridad de los datos consultando el conteo de registros en la tabla principal:

```bash
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT COUNT(*) AS total_peliculas FROM film;"
```

Salida:
```
 total_peliculas 
-----------------
            1000
(1 row)
```

Pruebe una consulta relacional que involucra múltiples tablas:

```bash
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT f.title, a.first_name || ' ' || a.last_name AS actor FROM film f JOIN film_actor fa ON f.film_id = fa.film_id JOIN actor a ON fa.actor_id = a.actor_id LIMIT 3;"
```

Salida:
```
        title         |      actor       
----------------------+------------------
 ACADEMY DINOSAUR     | PENELOPE GUINESS
 ANACONDA CONFESSIONS | PENELOPE GUINESS
 ANGELS LIFE          | PENELOPE GUINESS
(3 rows)
```

#### 9.3. Registro del servidor en pgAdmin desde el anfitrión

Considere que `pgadmin` se encuentra instalado en la máquina virtual y se accede a éste mediante un túnel SSH, tal como ya se configuró anteriormente.

Dentro de la interfaz web de pgAdmin accedida desde el navegador del anfitrión, registre el servidor de bases de datos utilizando la configuración del túnel:

1. En el panel izquierdo, haga clic derecho sobre **Servers** y seleccione **Register** > **Server**.
2. En la pestaña **General**, asigne un nombre descriptivo como `Fedora-Lab-Pagila`.
3. En la pestaña **Connection**, configure los siguientes parámetros:
   - Host name/address: `localhost`
   - Port: `5433`
   - Maintenance database: `pagila`
   - Username: `alumno`
   - Password: `uamIztapalapa`
   - Save password: desmarcado
4. En la pestaña **Parameters**, establezca SSL mode en `prefer` y Connection timeout en `10`.
5. Haga clic en **Save**.

Si la conexión es exitosa, el árbol de servidores mostrará la base de datos `pagila` y sus esquemas. Abra la herramienta **Query Tool** y ejecute las siguientes consultas para validar la integridad del entorno:

```sql
SELECT version();
SELECT COUNT(*) AS total_films FROM film;
SELECT f.title, f.release_year, a.first_name, a.last_name
FROM film f
JOIN film_actor fa ON f.film_id = fa.film_id
JOIN actor a ON fa.actor_id = a.actor_id
LIMIT 5;
```

Los resultados deben coincidir con la versión de PostgreSQL, un conteo de 1000 registros en la tabla `film` y un listado válido de películas y actores.

#### 9.4. Registro del servidor en DBeaver desde el anfitrión

[Dbeaver Community Edition](https://dbeaver.io/about/) es una herramienta de administración de base de datos libre y de código abierto para proyectos personales. Instalar [Dbeaver](https://dbeaver.io/about/) en su equipo Fedora 44 es muy sencillo instalando el paquete rpm de manera directa o usando flatpak:
```bash
sudo dnf install https://dbeaver.io/files/dbeaver-ce-latest-linux-x86_64.rpm
# o
flatpak install flathub io.dbeaver.DBeaverCommunity
```

Cuando ya se tiene `DBeaver` instalado y ejecutándose en su máquina virtual, este se configura de manera semejante a `pgAdmin`, tal como se abordó en la sección anterior. 

Ahora, abordaremos otro caso de uso en que `DBeaver` está instalado y ejecutándose en su sistema anfitrión y no en la máquina virtual y desde ahí se conecta a su base de datos Pagila. A continuación, se detallan las instrucciones formales y paso a paso para configurar `DBeaver` en su sistema local Fedora 44, con el objetivo de establecer una conexión segura hacia una base de datos postgreSQL Pagila alojada en su máquina virtual. 

Dado que la base de datos en la máquina virtual está restringida para aceptar únicamente conexiones desde su propio `localhost`, utilizaremos un túnel SSH para redirigir el tráfico de manera transparente en DBeaver.

**Paso 1: Iniciar la configuración de la conexión**  

1. Abra la aplicación **DBeaver** en su máquina local Fedora 44.  
2. En el menú superior, haga clic en **Base de datos** y seleccione **Nueva conexión...** (o haga clic en el icono de "Enchufe" con un asterisco en la barra de herramientas superior izquierda).  
3. En la ventana emergente, seleccione **PostgreSQL** y haga clic en **Siguiente**.  

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/dbeaver_01_nueva-conexion-a-bd.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 2. Crear nueva conexión a base de datos con DBeaver</div>
</div>

**Paso 2: Configurar los parámetros principales (Base de datos)**

En la pestaña **Principal**, ingrese los siguientes datos para la conexión a la base de datos:

*   **Servidor (Host):** `localhost` 
    *(Nota técnica: Es fundamental dejar `localhost` y no la IP de la máquina virtual. El túnel SSH que configuraremos más adelante se encargará de "hacer creer" a PostgreSQL que la conexión se origina desde su propio localhost).*  
*   **Puerto:** `5432` *(Puerto predeterminado de PostgreSQL)*.  
*   **Base de datos:** `pagila`  
*   **Nombre de usuario:** `alumno`  
*   **Contraseña:** `uamIztapalapa`  

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/dbeaver_02_configuracion-de-bd.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 3. Configuración de acceso a la base de datos con DBeaver</div>
</div>

**Paso 3: Configurar el Túnel SSH**

Para atravesar la restricción de red y conectarse a la máquina virtual, proceda a configurar el túnel:

1. Dentro de la misma ventana de configuración, haga clic en la pestaña **SSH** (ubicada en la parte superior o lateral, dependiendo de su versión de DBeaver).  
2. Marque la casilla **Usar túnel SSH** (Use SSH tunnel).  
3. Complete los siguientes campos para la conexión SSH:  
   *   **Host/IP:** `192.168.122.24`  
   *   **Puerto:** `22`  
   *   **Usuario:** `alumno`  
   *   **Método de autenticación:** Seleccione **Clave pública** (o *Key file*).  
   *   **Archivo llave (Key file):** Haga clic en el botón de explorar y seleccione la ruta de su llave privada. La ruta absoluta será: `/home/<su_usuario_local>/.ssh/fedora-lab-key` *(reemplace `<su_usuario_local>` con su nombre de usuario real en su Fedora local.*
   *   **Contraseña (Passphrase):** Si la llave privada `fedora-lab-key` está protegida con una frase de paso o si el servidor SSH requiere una contraseña como método alternativo/complementario, ingrese: `uamIztapalapa`. *(Si la llave no tiene passphrase, como en nuestro caso, el servidor acepta la llave sin más, puede dejar este campo vacío).*

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/dbeaver_03_configuracion-de-tunel-ssh.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 4. Configuración de túnel SSH a la máquina virtual con DBeaver</div>
</div>

**Paso 4: Probar y guardar la conexión**

1. Una vez completados los campos de la pestaña Principal y SSH, haga clic en el botón **Probar conexión** (Test Connection) en la esquina inferior izquierda.  
2. Si la configuración es correcta, DBeaver le mostrará un mensaje de éxito con los detalles de la conexión (versión de PostgreSQL, tiempo de respuesta, etc.) ().  
3. Haga clic en **Aceptar** o **Finalizar** para guardar la configuración.  

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/dbeaver_04_prueba-de-connection-a-bd.png" width="30%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 5. Prueba de conexión a base de datos Pagila con DBeaver</div>
</div>

La conexión aparecerá ahora en el panel de "Navegador de base de datos" a la izquierda. Al hacer doble clic sobre ella, DBeaver establecerá el túnel SSH hacia `192.168.122.24` y conectará el cliente local al puerto `5432` de la máquina virtual de manera segura y transparente.

<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/dbeaver_05_conexion-a-pagila.png" width="50%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Figura 6. Conexión a base de datos Pagila en DBeaver</div>
</div>


#### 10. Script de auditoría de seguridad

Para verificar de manera sistemática el cumplimiento de los controles implementados, utilice el siguiente script de validación. Este instrumento evalúa el estado de SELinux, contextos de archivos, permisos, configuración de Apache, reglas de cortafuegos (firewall) y estado de servicios.

Guarde el contenido en `validate_pgadmin_security.sh` y asigne permisos de ejecución:

```bash
#!/bin/bash
set -euo pipefail

GREEN='\033[0;32m'; RED='\033[0;31m'; YELLOW='\033[1;33m'; CYAN='\033[0;36m'; NC='\033[0m'
pass() { echo -e "${GREEN}[OK]${NC} $1"; }
fail() { echo -e "${RED}[ERROR]${NC} $1"; ((FAILS++)); }
info() { echo -e "${YELLOW}[INFO]${NC} $1"; }
header() { echo -e "\n${CYAN}--- $1 ---${NC}"; }

FAILS=0
echo -e "\n${CYAN}Validación de seguridad: pgAdmin 4 (Fedora 44)${NC}\n"

header "1. Modo SELinux"
[[ "$(getenforce)" == "Enforcing" ]] && pass "SELinux en modo Enforcing" || fail "SELinux en modo $(getenforce)"

header "2. Contextos de seguridad"
LOG_CTX=$(sudo ls -ldZ /var/log/pgadmin 2>/dev/null | awk '{print $5}')
LIB_CTX=$(sudo ls -ldZ /var/lib/pgadmin 2>/dev/null | awk '{print $5}')
[[ "$LOG_CTX" == *httpd_log_t* ]] && pass "Logs: $LOG_CTX" || fail "Logs: $LOG_CTX (esperado: httpd_log_t)"
[[ "$LIB_CTX" == *httpd_sys_rw_content_t* ]] && pass "Datos: $LIB_CTX" || fail "Datos: $LIB_CTX (esperado: httpd_sys_rw_content_t)"

header "3. Propiedad y permisos"
LOG_PERM=$(stat -c '%U:%G %a' /var/log/pgadmin 2>/dev/null)
LIB_PERM=$(stat -c '%U:%G %a' /var/lib/pgadmin 2>/dev/null)
[[ "$LOG_PERM" == "apache:apache 750" ]] && pass "Logs: $LOG_PERM" || info "Logs: $LOG_PERM"
[[ "$LIB_PERM" == "apache:apache 750" ]] && pass "Datos: $LIB_PERM" || info "Datos: $LIB_PERM"

header "4. Configuración de Apache"
grep -q "Require ip" /etc/httpd/conf.d/pgadmin4.conf 2>/dev/null && pass "Directiva Require ip activa" || fail "Falta directiva Require ip"

header "5. Cortafuegos firewalld"
if command -v firewall-cmd &> /dev/null; then
    sudo firewall-cmd --list-services 2>/dev/null | grep -q http && pass "Servicio HTTP permitido" || info "HTTP no listado en firewall"
    sudo firewall-cmd --list-ports 2>/dev/null | grep -q 5432/tcp && info "Puerto 5432 expuesto directamente" || pass "Puerto 5432 cerrado a red externa"
else
    info "firewalld no disponible"
fi

header "6. Booleanes SELinux"
[[ "$(getsebool httpd_can_network_connect 2>/dev/null | awk '{print $NF}')" == "on" ]] && pass "httpd_can_network_connect = on" || fail "Booleano desactivado"

header "7. Servicios críticos"
systemctl is-active --quiet httpd && pass "Apache activo" || fail "Apache inactivo"
systemctl is-active --quiet postgresql && pass "PostgreSQL activo" || info "PostgreSQL: $(systemctl is-active postgresql 2>/dev/null)"

echo -e "\n${CYAN}--- Reporte final ---${NC}"
[[ $FAILS -eq 0 ]] && echo -e "${GREEN}Todos los controles críticos están operativos.${NC}" || echo -e "${RED}Se detectaron $FAILS fallos. Revise las secciones marcadas.${NC}"
```

Ejecute la auditoría con privilegios elevados:

```bash
chmod +x validate_pgadmin_security.sh
sudo ./validate_pgadmin_security.sh
```

Un resultado sin fallos confirma que el entorno cumple con los estándares de seguridad definidos para el laboratorio.

#### 11. Consideraciones operativas y cierre

La arquitectura implementada separa claramente las capas de administración y datos. pgAdmin opera como interfaz web accesible mediante HTTP, mientras PostgreSQL permanece aislado y solo alcanzable a través de un túnel SSH cifrado. Esta separación reduce la superficie de ataque y facilita la auditoría de accesos.

Para finalizar las sesiones de trabajo, cierre el túnel SSH desde el anfitrión y detenga la máquina virtual si no se requiere uso inmediato:

```bash
### Finalizar el proceso que mantiene activo el túnel desde localhost hacia la VM
pkill -f "ssh -L 5433:localhost:5432"

### Apagar la máquina virtual
sudo virsh shutdown fedora44-lab
```

Mantenga actualizados los paquetes del sistema mediante `sudo dnf update` y rote las credenciales de acceso al concluir el período académico. Documente cada intervención en la bitácora de laboratorio, vinculando las configuraciones aplicadas con los criterios de evaluación de la unidad de enseñanza-aprendizaje.

#### 12. Referencias

DBeaver Corp. (2026). *DBeaver Community Edition* [Software]. <https://dbeaver.io/>
Fedora Project. (n.d.). *Fedora documentation* [Documentación de software]. <https://docs.fedoraproject.org/es_419/docs/>
OpenBSD Project. (n.d.). ssh(1). *OpenBSD manual pages* [Documentación de software]. <https://man.openbsd.org/ssh>
pgAdmin Development Team. (s. f.). *pgAdmin 4 documentation* [Documentación de software]. <https://www.pgadmin.org/docs/>
PostgreSQL Global Development Group. (2026). *PostgreSQL 18 documentation* [Documentación de software]. <https://www.postgresql.org/docs/18/>
Red Hat, Inc. (n.d.). Using SELinux. *Red Hat Enterprise Linux 9 documentation* [Documentación de software]. <https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/index>
