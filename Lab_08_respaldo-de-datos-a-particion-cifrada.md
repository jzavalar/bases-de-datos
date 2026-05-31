### Laboratorio 08. Respaldo de Datos a Partición Cifrada en Fedora
**(Opcional, pero recomendado)**

**Autor:** Dr. Jesús Zavala Ruiz  
**Última modificación:** 30 de Mayo de 2026  

---

#### I. Introducción

En la administración contemporánea de sistemas Linux, la protección de la información no constituye un complemento opcional, sino un requisito estructural derivado de la exposición a fallos de hardware, errores operativos y vectores de acceso no autorizado. Este laboratorio presenta una aproximación sistemática y reproducible a la implementación de un volumen de respaldo cifrado mediante el estándar **Linux Unified Key Setup 2** (**LUKS2**), integrado al ciclo de arranque de Fedora y gestionado mediante herramientas nativas de línea de comandos. 

El enfoque pedagógico prioriza la comprensión causal de cada intervención técnica: se explica no únicamente la sintaxis del comando, sino su posición dentro de la cadena de inicio del sistema, su impacto en la gestión de recursos y su relación con los mecanismos de seguridad del kernel. Asimismo, se incorpora el uso de `tmux` para garantizar la resiliencia de operaciones de larga duración, una práctica administrativa esencial en entornos donde las interrupciones de sesión, cortes de red o suspensiones del sistema son frecuentes. 

Al concluir este laboratorio, el alumno habrá adquirido competencia técnica en cifrado de bloques, persistencia de configuración de almacenamiento y sincronización segura de datos, consolidando un flujo de trabajo aplicable directamente en contextos institucionales y académicos.

#### II. Entorno de trabajo

El entorno requiere la disposición de un sistema operativo Fedora con un disco duro o de  estado sólido.

| Componente | Especificación |
|------------|----------------|
| **Sistema operativo** | Fedora Workstation (versión reciente) |
| **Usuario de trabajo** | `alumno` |
| **Almacenamiento** | Disco SSD único (`/dev/sda`), particionado para sistema operativo y espacio de datos |
| **Partición destino** | `/dev/sda4` (espacio sin formatear, ~674 GiB) |
| **Herramientas principales** | `gparted`, `cryptsetup`, `lsblk`, `dracut`, `rsync`, `tmux`, `systemd` |

> **Nota de diseño:**  
> El laboratorio parte de la premisa de un disco único que alberga el sistema operativo y requiere la creación de un volumen dedicado para respaldos, dejando el espacio para la instalación o reinstalación del sistema operativo Fedora Linux. Esta configuración es habitual en equipos de laboratorio o estaciones de trabajo personales, donde se busca optimizar el medio sin depender de dispositivos externos ni infraestructura de red.

#### III. Fase 1: Gestión y preparación del almacenamiento  

Antes de aplicar cualquier capa de cifrado, es indispensable disponer de un bloque de almacenamiento crudo, libre de metadatos y sin intervención del sistema de archivos activo. GParted permite modificar la geometría del disco de manera segura, siempre que se respeten los protocolos de precaución.

> **Advertencia crítica:**
> Las operaciones de particionamiento alteran la tabla de particiones. Aunque la aplicación GParted, en Gnome, está diseñada para preservar datos existentes, una interrupción eléctrica o un cierre forzado durante el redimensionamiento puede corromper el sistema de archivos.
> **Se requiere un respaldo previo de toda información crítica antes de ejecutar este paso.**

#### 1. Procedimiento

1. Ejecute `sudo gparted` desde la sesión instalada o desde un Live USB (evitando modificar la partición raíz activa).
2. Seleccione `/dev/sda` en el menú superior derecho.
3. Libere espacio, haciendo click derecho sobre la partición adyacente → **Redimensionar/Mover**. Ajuste el tamaño para dejar ~690 101 MiB libres, suficiente para realizar el respaldo de datos completo (como en este caso) o de una dimensión más pequeña, por ejemplo, de 4 096 MiB, para realizar una demostración. 
4. En el espacio no asignado, haga clic derecho → **Nuevo**. Configure:
   - **Sistema de archivos:** `unformatted` (**indispensable**)
   - **Etiqueta:** `datos`
   - **Alinear a:** `MiB` (garantiza compatibilidad óptima con SSD)
5. Presione **✓ Aplicar** y espere la finalización sin interrupciones.

#### 2. Verificación inicial

```bash
$ lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINT /dev/sda
NAME                                    SIZE FSTYPE      LABEL  MOUNTPOINT
sda                                   931.5G                    
├─sda1                                  600M vfat               /boot/efi
├─sda2                                    1G ext4               /boot
├─sda3                                  256G crypto_LUKS        
│ └─luks-197405dc-ce24-4896-8658-e0abb0d620f8
│                                       256G btrfs       fedora /home
│                                                                            /
└─sda4                                673.9G                    
```

> **Interpretación:**  
> La ausencia de `FSTYPE` y de punto de montaje en `/dev/sda4` confirma que se trata de un bloque de almacenamiento sin estructura lógica, condición ideal para la inicialización criptográfica.

#### IV. Fase 2: Implementación del cifrado LUKS2 y sistema de archivos

LUKS (Linux Unified Key Setup) estandariza el cifrado de bloques en Linux. La versión 2 introduce `argon2id` para la derivación de claves (resistente a ataques por fuerza bruta con GPU) y `aes-xts-plain64` como modo de cifrado por defecto. Es fundamental comprender que *LUKS no cifra los datos directamente*, sino que protege una *clave maestra* almacenada en la cabecera del volumen; la frase contraseña que ingresa el usuario sirve únicamente para desbloquear dicha clave.

#### 1. Inicialización del contenedor LUKS2

```bash
$ sudo cryptsetup luksFormat --type luks2 --label datos /dev/sda4

¡ATENCIÓN!
==========
Esto sobreescribirá los datos en /dev/sda4 de forma irrevocable.

¿Está seguro? (Teclee 'yes' en mayúsculas): YES
Introduzca la frase contraseña de /dev/sda4: 
Verifique la frase contraseña: 
```

> **Advertencia:**
> Guarde y recuerde muy bien la **frase contraseña** ya que si la píerde, será **imposible** abrir el volumen encriptado.

#### 2. Apertura del volumen y creación del mapeador

Un dispositivo LUKS cerrado es ilegible para el sistema. `luksOpen` descifra la clave maestra en memoria RAM y expone un dispositivo virtual en `/dev/mapper/`. Este mapeador actúa como un filtro transparente: todo dato escrito se cifra antes de alcanzar el disco físico; todo dato leído se descifra antes de llegar a la aplicación.

```bash
$ sudo cryptsetup luksOpen /dev/sda4 datos
Introduzca la frase contraseña de /dev/sda4: 
$ lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINT | grep -E "sda4|datos"
└─sda4                                        673.9G crypto_LUKS datos  
  └─datos                                     673.9G                    
```

#### 3. Formateo del sistema de archivos `ext4`

Se formatea `/dev/mapper/datos`, no `/dev/sda4`, con us sistema de archivos  `ext4` por su estabilidad, bajo consumo de memoria y compatibilidad probada con operaciones `discard` (TRIM) en el disco de estado sólido (SSD).

```bash
$ sudo mkfs.ext4 -L datos_vol /dev/mapper/datos
mke2fs 1.47.2 (1-Jan-2025)
Se está creando un sistema de ficheros con 176661504 bloques de 4k y 44171264 nodos-i
UUID del sistema de ficheros: 2a3e2d53-5f51-446a-b762-a693a26536fd
Respaldos del superbloque guardados en los bloques: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000

Reservando las tablas de grupo: hecho                           
Escribiendo las tablas de nodos-i: hecho                           
Creando el fichero de transacciones (262144 bloques): hecho
Escribiendo superbloques y la información contable del sistema de ficheros: hecho    
```

#### V. Fase 3: Configuración de persistencia y montaje automático

Para que el volumen sobreviva a reinicios, `systemd` debe conocer dos parámetros fundamentales: *cómo abrir el cifrado* y *dónde montarlo*. Esta información se registra en `/etc/crypttab` y `/etc/fstab`, respectivamente.

#### 1. Obtención del identificador único (UUID)

Antes de registrar el volumen cifrado en los archivos de configuración del sistema, es indispensable obtener su UUID. Este identificador es generado aleatoriamente durante la inicialización del contenedor LUKS y permanece inmutable a lo largo de la vida útil del dispositivo, incluso si el kernel reordena la nomenclatura de los bloques (ej. `/dev/sda4` → `/dev/sdb1` tras conectar un disco externo).

**Instrucción:**

```bash
$ sudo blkid /dev/sda4
/dev/sda4: UUID="54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7" TYPE="crypto_LUKS" LABEL="datos" PARTUUID="c607c2f0-fc0d-43e4-960d-ca538edfff6c"
```

El valor que figura tras `UUID=` corresponde al identificador de la partición física cifrada. Debe copiarse íntegramente (incluidas las comillas o sin ellas, según la sintaxis de `crypttab`) para ser insertado en dicho archivo. `systemd-cryptsetup` utiliza este UUID como referencia absoluta durante la fase de `initramfs`, garantizando que el mapeador se asocie correctamente al dispositivo incluso en entornos con múltiples discos o controladores de almacenamiento de distinta arquitectura. Su uso descarta la ambigüedad inherente a las rutas `/dev/sdX`, las cuales son volátiles y dependen del orden de inicialización del kernel en cada arranque.

#### 2. Edición de archivos de configuración

Abra **`/etc/crypttab`** (desencriptación en arranque) y agregue la siguiente línea, al final del archivo:

```
datos UUID=54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7 none luks,discard
```
- `datos`: nombre del mapeador en `/dev/mapper/`.
- `UUID=...`: identificador inmutable de la partición física.
- `none`: indica que Plymouth solicitará la frase interactivamente durante el inicio.
- `luks,discard`: activa el protocolo LUKS2 y permite que las instrucciones TRIM del SSD atraviesen la capa cifrada, preservando el rendimiento y la vida útil del medio.

Luego, abra **`/etc/fstab`** (montaje del sistema de archivos) y agregue la siguiente línea, al final del archivo:

```
/dev/mapper/datos /backup ext4 defaults,noatime 0 2
```
- `noatime`: evita registrar timestamps de lectura en cada acceso, reduciendo escrituras innecesarias y mejorando el rendimiento en SSD.
- `0 2`: omite respaldos con `dump`, pero permite verificación de integridad con `fsck` tras el montaje de la raíz.

#### 3. Regeneración de `initramfs` y aplicación de cambios

El kernel carga un sistema mínimo en memoria (`initramfs`) antes de montar la partición raíz. `dracut` reconstruye esta imagen para incluir `systemd-cryptsetup` y leer `crypttab`. Posteriormente, `systemd` debe recargar su configuración para reconocer `fstab`.

```bash
$ sudo dracut -f
$ sudo mkdir -p /backup
$ sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
$ systemctl daemon-reload
$ sudo mount -a
$ df -h /backup
S.ficheros        Tamaño Usados  Disp Uso% Montado en
/dev/mapper/datos   663G   2.1M  629G   1% /backup
$ sudo chown $USER:$USER /backup
$ ls -ld /backup
drwxr-xr-x 3 alumno alumno 4096 may 29 14:17 /backup
```

Tras recargar `systemd`, el volumen se montó correctamente. La asignación de propiedad garantiza que el usuario `alumno` opere con permisos de lectura/escritura sin requerir privilegios de superusuario.

#### VI. Fase 4: Ejecución resiliente del respaldo con `tmux` y `rsync`
Un respaldo de ~150 GiB puede extenderse entre 30 y 90 minutos. Si la terminal se cierra, la conexión SSH se interrumpe o el sistema suspende la sesión, `rsync` se terminará abruptamente, corrompiendo potencialmente la copia. `tmux` (terminal multiplexer) ejecuta procesos en sesiones virtuales independientes de la terminal física, permitiendo desconectarse y reconectarse sin perder el progreso ni el contexto de ejecución.

#### 1. Sesión resiliente con `tmux`

```bash
### 1. Crear y adjuntar una sesión persistente llamada "respaldo"
$ tmux new-session -d -s respaldo
$ tmux attach -t respaldo

### 2. Dentro de tmux, preparar directorio y ejecutar rsync
$ BACKUP_DIR="/backup/respaldos_home/$(date +%Y%m%d)"
$ sudo mkdir -p "$BACKUP_DIR"
$ sudo rsync -aAXHv --progress \
  --exclude='.cache' \
  --exclude='.local/share/Trash' \
  --exclude='.thumbnails' \
  --exclude='.gvfs' \
  /home/alumno/ "$BACKUP_DIR/"
```
> **Desvinculación segura sin detener el proceso:**
> Presione `Ctrl+B`, suelte ambas teclas y presione `D`. La terminal se liberará, pero `rsync` continuará ejecutándose en segundo plano dentro de `tmux`.
> 
> **Reconexión o recuperación de la sesión:**
> ```bash
> $ tmux attach -t respaldo
> ```
>
> El usuario recupera la consola exactamente en el punto donde quedó, sin interrupciones ni pérdida de contexto.

#### 2. Salida final del proceso de respaldo
```
sent 166,565,358,492 bytes  received 597,485 bytes  39,672,729.78 bytes/sec
total size is 166,522,513,580  speedup is 1.00
```

La velocidad promedio (~39.7 MB/s) es coherente con la sobrecarga mínima introducida por el cifrado LUKS2. `tmux` garantizó la ejecución ininterrumpida, demostrando su valor en operaciones de mantenimiento crítico.

#### 3. Salida de la sesión resiliente `tmux`

> ```bash
> $ exit
> ```

#### VII. Fase 5: Validación integral y prueba de ciclo completo

La validación definitiva consiste en cerrar el volumen, reiniciar el equipo y verificar que el sistema solicita la frase de cifrado durante el arranque, desencripta el medio y lo monta automáticamente.

#### 1. Preparación y reinicio

```bash
$ sudo umount /backup
$ sudo cryptsetup luksClose datos
$ sudo reboot
```

#### 2. Validación posterior al reinicio

```bash
$ df -h /backup
S.ficheros        Tamaño Usados  Disp Uso% Montado en
/dev/mapper/datos   663G   156G  474G  25% /backup

$ lsblk -o NAME,FSTYPE,MOUNTPOINT | grep -E "sda4|datos"
└─sda4                                        crypto_LUKS 
  └─datos                                     ext4        /backup

$ systemctl status systemd-cryptsetup@datos.service --no-pager -l
● systemd-cryptsetup@datos.service - Cryptography Setup for datos
     Loaded: loaded (/etc/crypttab; generated)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (exited) since Fri 2026-05-29 22:39:45 CST; 14min ago
 Invocation: d45a574737574f2a8935d58a3b54f43d
       Docs: man:crypttab(5)
             man:systemd-cryptsetup-generator(8)
             man:systemd-cryptsetup@.service(8)
    Process: 82854 ExecStart=/usr/bin/systemd-cryptsetup attach datos /dev/disk/by-uuid/54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7 none luks,discard (code=exited, status=0/SUCCESS)
   Main PID: 82854 (code=exited, status=0/SUCCESS)
   Mem peak: 827.1M
        CPU: 5.543s

may 29 22:39:41 fedora systemd[1]: Starting systemd-cryptsetup@datos.service - Cryptography Setup for datos...
may 29 22:39:45 fedora systemd[1]: Finished systemd-cryptsetup@datos.service - Cryptography Setup for datos.

$ sudo grep datos /etc/crypttab
datos UUID=54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7 none luks,discard
$ sudo grep backup /etc/fstab
/dev/mapper/datos /backup ext4 defaults,noatime 0 2
```

> **Conclusión de la prueba:**
> El servicio `systemd-cryptsetup` desencriptó el volumen en fase temprana de arranque, `fstab` lo montó automáticamente y los datos son accesibles. La configuración es persistente, funcional y lista para producción.

#### VIII. Registro técnico consolidado

| Parámetro | Valor configurado |
|-----------|-------------------|
| **Dispositivo físico** | `/dev/sda4` (673.9 GiB) |
| **Etiqueta LUKS** | `datos` |
| **UUID LUKS** | `54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7` |
| **Nombre de mapeo** | `datos` → `/dev/mapper/datos` |
| **Sistema de archivos** | `ext4` (etiqueta: `datos_vol`) |
| **Punto de montaje** | `/backup` |
| **Opciones `crypttab`** | `none luks,discard` |
| **Opciones `fstab`** | `defaults,noatime 0 2` |
| **Datos respaldados** | `~156 GiB` en `/backup/respaldos_home/20260529/` |
| **Estado operativo** | ✅ Validado en ciclo completo de arranque |

#### IX. Conclusiones

Este laboratorio demuestra que la seguridad de datos y la automatización del arranque no requieren infraestructura compleja ni soluciones propietarias. Mediante herramientas nativas de Fedora (`cryptsetup`, `systemd`, `dracut`, `rsync`), es posible construir un flujo de respaldo cifrado, persistente y resiliente a interrupciones de sesión. La integración de `tmux` ilustra una práctica administrativa fundamental: desacoplar la ejecución de procesos críticos de la estabilidad de la terminal cliente, garantizando la finalización exitosa de operaciones de larga duración sin supervisión constante.

El esquema resultante cumple con los principios de confidencialidad (cifrado LUKS2 con `argon2id`), integridad (preservación de metadatos, ACLs y contextos SELinux mediante `rsync -aAXHv`) y disponibilidad (montaje automático gestionado por `systemd` y validación cíclica). Su implementación en entornos académicos e institucionales no solo protege la información sensible, sino que establece una base reproducible para la adopción de prácticas de ciberseguridad alineadas con estándares internacionales. Se recomienda complementar este flujo con políticas de retención automatizada, verificación de integridad mediante checksums y respaldo periódico de la cabecera LUKS en medios externos seguros.

#### X. Referencias

Anónimo. (2024). Capítulo 8. Cifrado de dispositivos de bloque mediante LUKS In *Red Hat Enterprise Linux 8: Seguridad de Red Hat Enterprise Linux 8*. Red Hat. <https://docs.redhat.com/es/documentation/red_hat_enterprise_linux/8/html-single/security_hardening/index#encrypting-block-devices-using-luks_security-hardening>

Anónimo (2025, mayo 22). Encriptación de disco completo con LUKS. <https://linuxmind.dev/es/2025/05/22/encriptacion-de-disco-completo-con-luks/>

Anónimo. (2025). cryptsetup(8), systemd-cryptsetup-generator(8), crypttab(5) y fstab(5) manual pages. <https://manpages.org/cryptsetup/8>, <https://manpages.org/systemd-cryptsetup-generator/8>, <https://es.manpages.org/crypttab/5>, <https://es.manpages.org/fstab/5>

Anónimo. (2026, May 8). *rsync -⁠ a fast, versatile, remote (and local) file-copying tool*. <https://manpages.org/rsync>, <https://rsync.samba.org/>

Nicholas Marriott, N. (2025, Dec 22). *tmux: Terminal multiplexer documentation*. <https://github.com/tmux/tmux/wiki>


Ziegler, M. (2025). *dracut: Initramfs infrastructure for modern Linux distributions*. https://dracut.wiki.kernel.org/
