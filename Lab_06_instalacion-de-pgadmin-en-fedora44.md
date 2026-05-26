### Laboratorio 06. Instalación de pgAdmin en Fedora 44
**Unidad de Enseñanza-Aprendizaje:** Bases de Datos (2151106)  
**Institución:** Universidad Autónoma Metropolitana, Unidad Iztapalapa  

**Autor:** Dr. Jesús Zavala Ruiz  
**Fecha:** Mayo 2026  
**Entorno de ejecución:** Fedora 44 Cloud Base sobre KVM/libvirt  

**Dirección de la máquina virtual:** 192.168.122.24  
**Usuario de acceso:** alumno  

---

#### Introducción

La administración eficiente de sistemas gestores de bases de datos requiere herramientas que simplifiquen la interacción con el motor, permitan visualizar esquemas complejos y faciliten la ejecución de consultas sin depender exclusivamente de interfaces de línea de comandos. pgAdmin 4 constituye la plataforma de administración gráfica oficial para PostgreSQL, ofreciendo una arquitectura web que permite gestionar múltiples instancias de bases de datos desde un navegador estándar.

En este laboratorio se documenta el procedimiento completo para desplegar pgAdmin 4 en modalidad web sobre Fedora 44, integrando los componentes necesarios, aplicando controles de seguridad obligatoria mediante SELinux, configurando el acceso restringido a través de Apache y estableciendo una capa de filtrado de red con firewalld. La arquitectura resultante replica prácticas estándar de hardening utilizadas en entornos productivos, garantizando que la interfaz de administración permanezca accesible mientras el motor de bases de datos se mantiene aislado y protegido.

#### 1. Requisitos del entorno

Antes de iniciar, confirme que la máquina virtual cumple con las siguientes condiciones:

- Máquina virtual (VM) `fedora44-lab` con sistema operativo Fedora 44 Cloud Base actualizado.
- Usuario `alumno` con privilegios de administración mediante sudo.
- PostgreSQL 18 operativo con la base de datos `pagila` cargada.
- Clave SSH configurada para acceso remoto desde el equipo anfitrión.
- Conexión de red tipo NAT o puente gestionada por `libvirt`.

#### 2. Instalación de componentes base en la Máquina Virtual

**2.1. Iniciar la VM:**

```bash
sudo virsh start fedora44-lab
```

Salida:
```
[sudo] password for jzavalar: 
Domain 'fedora44-lab' started
```

**2.2. Conectarse a la VM:**
```bash
# Conectarse a la VM
ssh -i ~/.ssh/fedora-lab-key alumno@192.168.122.24
```

**2.3. Instalación de componentes base en la VM**

El primer paso consiste en sincronizar los repositorios del sistema y adquirir los paquetes que conforman la pila de pgAdmin en Fedora. Dado que pgAdmin está desarrollado en Python y requiere un servidor web para su ejecución en modalidad remota, es necesario instalar simultáneamente la aplicación, su módulo de integración con Apache y el propio servidor HTTP.

Ejecute los siguientes comandos dentro de la máquina virtual:

```bash
sudo dnf update -y
sudo dnf install -y pgadmin4 pgadmin4-httpd httpd
rpm -q pgadmin4 httpd
```

La salida esperada confirma las versiones instaladas:
```
pgadmin4-9.15-1.fc44.x86_64
httpd-2.4.67-1.fc44.x86_64
```

Una vez instalados los paquetes, es necesario habilitar el servicio de Apache para que inicie automáticamente con el sistema y activarlo de manera inmediata:

```bash
sudo systemctl enable --now httpd
sudo systemctl is-active httpd
sudo ss -tlnp | grep :80
```

La confirmación `active` y la línea `LISTEN ... *:80` indican que el servidor web está operando y recibiendo conexiones en el puerto estándar.

#### 3. Configuración del servidor web Apache y pgAdmin en la MV

Por defecto, la configuración de pgAdmin en Fedora restringe el acceso a la interfaz web únicamente desde la dirección local (`localhost`). Para permitir la conexión desde el equipo anfitrión a través de la red virtual, es necesario modificar la directiva de control de acceso en la configuración de Apache.

Se recomienda crear un respaldo antes de realizar cambios:

```bash
sudo cp /etc/httpd/conf.d/pgadmin4.conf /etc/httpd/conf.d/pgadmin4.conf.bak
sudo sed -i 's/Require local/Require ip 192.168.122.0\/24 127.0.0.1 ::1/' /etc/httpd/conf.d/pgadmin4.conf
grep "Require" /etc/httpd/conf.d/pgadmin4.conf
```

La directiva modificada autoriza explícitamente las solicitudes provenientes de la subred `192.168.122.0/24`, manteniendo al mismo tiempo el acceso local para fines de diagnóstico. Es importante notar que la ruta de acceso configurada es `/pgadmin`, por lo que la URL de acceso final será `http://192.168.122.24/pgadmin/`. Abra esa URL en un navegador, para acceder a pgAdmin desde el anfitrión:


<div style="text-align: center;">
  <img src="https://github.com/jzavalar/bases-de-datos/blob/main/imagenes/lab_06_pgadmin-desde-localhost.png" width="70%">
  <div style="font-size: 0.9em; margin-top: 0.5em;">Fig. 1. Acceso a pgAdmin desde el anfitrión</div>
</div>

#### 4. Inicialización de pgAdmin en la MV

La aplicación requiere una base de datos interna en formato SQLite para almacenar credenciales de usuarios, configuraciones de servidores registrados y preferencias de sesión. Fedora distribuye un script de inicialización basado en Python que debe ejecutarse con privilegios elevados.

Localice la ruta del script y consulte sus subcomandos disponibles:

```bash
SETUP_SCRIPT=$(rpm -ql pgadmin4 | grep '/setup\.py$' | head -n 1)
sudo python3 "$SETUP_SCRIPT" --help
```

Entre las opciones listadas, el subcomando `setup-db` es el responsable de crear la estructura de almacenamiento interno. Ejecute la inicialización:

```bash
sudo python3 "$SETUP_SCRIPT" setup-db
```

El proceso es idempotente y no produce salida visible si se completa correctamente. La ausencia de errores indica que la base de datos interna `pgadmin4.db` ha sido generada en `/var/lib/pgadmin/`.

#### 5. Ajuste de permisos y políticas SELinux en la MV

Fedora implementa Security-Enhanced Linux (SELinux) en modo obligatorio (`Enforcing`). Esta capa de seguridad evalúa no solo los permisos tradicionales del sistema de archivos, sino también los contextos de seguridad asociados a cada archivo y directorio. Si el proceso de Apache (`httpd_t`) intenta escribir en un directorio que posee un contexto genérico, la operación será bloqueada independientemente de los permisos `rwx`.

Para garantizar la operatividad, es necesario instalar las herramientas de gestión de políticas y aplicar los contextos específicos requeridos por la aplicación web:

```bash
sudo dnf install -y policycoreutils-python-utils
sudo mkdir -p /var/lib/pgadmin /var/log/pgadmin
sudo chown -R apache:apache /var/lib/pgadmin /var/log/pgadmin
sudo chmod -R 750 /var/lib/pgadmin /var/log/pgadmin
```

A continuación, defina y aplique los contextos de seguridad:

```bash
sudo semanage fcontext -a -t httpd_log_t "/var/log/pgadmin(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/lib/pgadmin(/.*)?"
sudo restorecon -Rv /var/log/pgadmin /var/lib/pgadmin
```

Verifique la aplicación correcta de los contextos:

```bash
sudo ls -laZ /var/log/pgadmin/
sudo ls -laZ /var/lib/pgadmin/
```

La salida debe mostrar `httpd_log_t` para los registros y `httpd_sys_rw_content_t` para los datos. Adicionalmente, active el booleano que permite a Apache establecer conexiones de red salientes, necesario para el funcionamiento de pgAdmin:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

#### 6. Configuración del cortafuegos (firewalld) en la MV

Las imágenes minimalistas de Fedora Cloud Base no incluyen el gestor de cortafuegos por defecto. Para completar el endurecimiento del entorno, instale y configure `firewalld`, garantizando que únicamente el tráfico HTTP autorizado pueda alcanzar la interfaz de administración.

Instale el servicio y actívelo:

```bash
sudo dnf install -y firewalld
sudo systemctl enable --now firewalld
```

Verifique que el servicio está operando y revise la zona activa:

```bash
sudo systemctl is-active firewalld
sudo firewall-cmd --list-all
```

Inicialmente, el servicio `http` no aparecerá en la lista. Agréguelo de forma permanente y recargue las reglas sin interrumpir servicios activos:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```

La salida debe incluir `http` junto a los servicios base como `ssh`. Confirme que no existen puertos expuestos de forma numérica directa:

```bash
sudo firewall-cmd --list-ports
```

Una lista vacía o sin referencias al puerto `5432` confirma que el motor de bases de datos permanece protegido detrás del túnel SSH, cumpliendo con el principio de mínimo privilegio.

#### 7. Validación de acceso web en la MV

Tras aplicar todas las configuraciones, reinicie el servidor web para consolidar los cambios y verifique que la aplicación responde correctamente a solicitudes HTTP:

```bash
sudo systemctl restart httpd
curl -s -o /dev/null -w "HTTP Code: %{http_code}\n" http://localhost/pgadmin/
curl -v http://localhost/pgadmin/ 2>&1 | grep -E "HTTP/|Location:"
```

Un código de respuesta `302` con redirección hacia `/pgadmin/login` confirma que la pila está operativa y que el sistema solicita autenticación, comportamiento esperado en la primera carga.

Desde el equipo anfitrión, confirme la conectividad externa:

```bash
curl -s -o /dev/null -w "HTTP Code: %{http_code}\n" http://192.168.122.24/pgadmin/
```

Acceda mediante un navegador web a la dirección `http://192.168.122.24/pgadmin/` y complete el registro de la cuenta maestra solicitada. Estas credenciales son exclusivas para la interfaz de pgAdmin y no guardan relación con el sistema operativo ni con PostgreSQL.

#### 8. Conexión segura a PostgreSQL en la MV mediante túnel SSH desde el anfitrión (localhost)

**8.1. Creación de túnel SSH en el anfitrión**

Por diseño, PostgreSQL escucha únicamente en la interfaz local de la máquina virtual. Para administrar la base de datos desde el equipo anfitrión sin exponer el puerto `5432` en la red, establezca un túnel SSH cifrado:

```bash
ssh -L 5433:localhost:5432 -i ~/.ssh/fedora-lab-key alumno@192.168.122.24 -N -f
ss -tlnp | grep 5433
```

El parámetro `-L 5433:localhost:5432` redirige el puerto local `5433` del anfitrión (localhost) hacia el puerto `5432` de la máquina virtual. La bandera `-f` ejecuta el proceso en segundo plano y `-N` inhibe la ejecución de comandos remotos. La presencia de `LISTEN ... 127.0.0.1:5433` confirma que el canal seguro está activo.

**8.2. Conexión segura a PostgreSQL en la MV mediante un túnel SSH desde localhost con el cliente psql**

```bash
# Verficar versión de PostgreSQL
psql --version
```

Salida:
```
psql (PostgreSQL) 16.13
```

```bash
# Verificar parametros de conexion a la base de datos
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT current_user, current_database(), inet_server_port();"
```

Salida:
```
 current_user | current_database | inet_server_port 
--------------+------------------+------------------
 alumno       | pagila           |             5432
(1 row)
```

```bash
# Verificar versión de PostgreSQL 
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT version();"
```

Salida:
```

                                                   version                          
                          
------------------------------------------------------------------------------------
--------------------------
 PostgreSQL 18.3 on x86_64-redhat-linux-gnu, compiled by gcc (GCC) 16.0.1 20260321 (
Red Hat 16.0.1-0), 64-bit
(1 row)
```

```bash
# Validar conexión a la base de datos con el conteo de registros en la tabla principal
psql -h localhost -p 5433 -U alumno -d pagila -c "SELECT COUNT(*) AS total_peliculas FROM film;"
```

Salida:
```
 total_peliculas 
-----------------
            1000
(1 row)
```

```bash
# Probar relación entre tablas mediante JOIN
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

**8.3. Conexión segura a PostgreSQL en la MV mediante un túnel SSH desde localhost con pgAdmin** 

Dentro de pgAdmin local, registre el servidor de bases de datos utilizando la configuración del túnel:

1. Navegue a Servers, haga clic derecho y seleccione Register > Server.
2. En la pestaña General, asigne un nombre descriptivo como `Fedora-Lab-Pagila`.
3. En la pestaña Connection, configure:
   - Host name/address: `localhost`
   - Port: `5433`
   - Maintenance database: `pagila`
   - Username: `alumno`
   - Password: `uamIztapalapa`
   - Save password: desmarcado
4. En la pestaña Parameters, establezca SSL mode en `prefer` y Connection timeout en `10`.
5. Guarde la configuración.

Si la conexión es exitosa, el árbol de servidores mostrará la base de datos `pagila` y sus esquemas. Abra la herramienta Query Tool y ejecute las siguientes consultas para validar la integridad del entorno:

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



#### Script de auditoría de seguridad de pgAdmin en la MV

Para verificar de manera sistemática el cumplimiento de los controles implementados, utilice el siguiente script de validación. Este instrumento evalúa el estado de SELinux, contextos de archivos, permisos, configuración de Apache, reglas de cortafuegos y estado de servicios.

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

#### Consideraciones operativas y cierre

La arquitectura implementada separa claramente las capas de administración y datos. pgAdmin opera como interfaz web accesible mediante HTTP, mientras PostgreSQL permanece aislado y solo alcanzaable a través de un túnel SSH cifrado. Esta separación reduce la superficie de ataque y facilita la auditoría de accesos.

Para finalizar las sesiones de trabajo, cierre el túnel SSH desde el anfitrión (localhost) y detenga la máquina virtual si no se requiere uso inmediato:

```bash
# Matar el proceso que mantiene activo el tunel del localhost a la MV
pkill -f "ssh -L 5433:localhost:5432"

# Apagar la VM
sudo virsh shutdown fedora44-lab
```

Mantenga actualizados los paquetes del sistema mediante `sudo dnf update` y rote las credenciales de acceso al concluir el período académico. Documente cada intervención en la bitácora de laboratorio, vinculando las configuraciones aplicadas con los criterios de evaluación de la unidad de enseñanza-aprendizaje.

#### Referencias

- pgAdmin 4 Documentation. <https://www.pgadmin.org/docs/>
- Fedora Project Documentation. <https://docs.fedoraproject.org/>
- PostgreSQL Documentation. <https://www.postgresql.org/docs/>
- Red Hat Enterprise Linux SELinux Guide. <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/>
- OpenSSH Manual Pages. <https://man.openbsd.org/ssh>
