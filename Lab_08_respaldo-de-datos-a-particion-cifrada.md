### Laboratorio 08. Respaldo de Datos a Partición Cifrada en Fedora
**(Opcional, pero recomendado)**

**Autor:** Dr. Jesús Zavala Ruiz\
**Última modificación:** 31 de Mayo de 2026

------------------------------------------------------------------------

#### I. Introducción

En la administración contemporánea de sistemas Linux, la protección de la información no constituye un complemento opcional, sino un requisito estructural derivado de la exposición a fallos de hardware, errores operativos y vectores de acceso no autorizado. Este laboratorio presenta una aproximación sistemática y reproducible a la implementación de un volumen de respaldo cifrado mediante el estándar **Linux Unified Key Setup 2** (**LUKS2**), integrado al ciclo de arranque de Fedora y gestionado mediante herramientas nativas de línea de comandos.

El enfoque pedagógico prioriza la comprensión causal de cada intervención técnica: se explica no únicamente la sintaxis del comando, sino su posición dentro de la cadena de inicio del sistema, su impacto en la gestión de recursos y su relación con los mecanismos de seguridad del kernel. Asimismo, se incorpora el uso de `tmux` para garantizar la resiliencia de operaciones de larga duración, una práctica administrativa esencial en entornos donde las interrupciones de sesión, cortes de red o suspensiones del sistema son frecuentes.

Al concluir este laboratorio, el alumno habrá adquirido competencia técnica en cifrado de bloques, persistencia de configuración de almacenamiento, sincronización segura de datos y recuperación ante fallos de metadatos criptográficos, consolidando un flujo de trabajo aplicable directamente en contextos institucionales y académicos.

#### II. Entorno de trabajo

El procedimiento requiere un sistema Fedora Workstation con acceso a un disco de estado sólido o magnético susceptible de particionamiento. La siguiente configuración establece los parámetros base sobre los cuales se desarrollará el laboratorio.

| Componente | Especificación |
|-------------------------------|-----------------------------------------|
| **Sistema operativo** | Fedora Workstation (versión reciente) |
| **Usuario de trabajo** | `alumno` |
| **Almacenamiento** | Disco SSD único (`/dev/sda`), particionado para sistema operativo y espacio de datos de 1 TB |
| **Partición destino** | `/dev/sda4` (espacio sin formatear, \~674 GiB) |
| **Herramientas principales** | `gparted`, `cryptsetup`, `lsblk`, `dracut`, `rsync`, `tmux`, `systemd`, `vi` |

> **Nota de diseño:**\
> El laboratorio parte de la premisa de un disco único que alberga el sistema operativo y requiere la creación de un volumen dedicado para respaldos, dejando espacio para la instalación o reinstalación del sistema operativo Fedora Linux. Esta configuración es habitual en equipos de laboratorio o estaciones de trabajo personales, donde se busca optimizar el medio sin depender de dispositivos externos ni infraestructura de red.

#### III. Fase 1: Gestión y preparación del almacenamiento

Antes de aplicar cualquier capa de cifrado, es indispensable disponer de un bloque de almacenamiento crudo, libre de metadatos y sin intervención del sistema de archivos activo. GParted permite modificar la geometría del disco de manera segura, siempre que se respeten los protocolos de precaución.

> **Advertencia crítica:**\
> Las operaciones de particionamiento alteran la tabla de particiones. Aunque GParted está diseñado para preservar datos existentes, una interrupción eléctrica o un cierre forzado durante el redimensionamiento puede corromper el sistema de archivos. **Se requiere un respaldo previo de toda información crítica antes de ejecutar este paso.**

#### 1. Procedimiento de particionamiento

1.  Ejecute `sudo gparted` desde la sesión instalada o desde un Live USB (evitando modificar la partición raíz activa).
2.  Seleccione `/dev/sda` en el menú superior derecho.
3.  Libere espacio haciendo clic derecho sobre la partición adyacente → **Redimensionar/Mover**. Ajuste el tamaño para dejar \~690 101 MiB libres, suficiente para realizar el respaldo de datos completo (como en este caso) o de una dimensión más pequeña, por ejemplo, de 4 096 MiB, si lo que quieres es realizar una demostración.
4.  En el espacio no asignado, haga clic derecho → **Nuevo**. Configure:
    -   **Sistema de archivos:** `unformatted` (**indispensable**)
    -   **Etiqueta:** `datos`
    -   **Alinear a:** `MiB` (garantiza compatibilidad óptima con SSD)
5.  Presione **✓ Aplicar** y espere la finalización sin interrupciones.

#### 2. Verificación inicial

``` bash
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

> **Interpretación:**\
> La ausencia de `FSTYPE` y de punto de montaje en `/dev/sda4` confirma que se trata de un bloque de almacenamiento sin estructura lógica, condición ideal para la inicialización criptográfica. Con el espacio preparado, se procede a aplicar la capa de cifrado que protegerá los datos del respaldo.

#### IV. Fase 2: Implementación del cifrado LUKS2 y sistema de archivos

LUKS (Linux Unified Key Setup) estandariza el cifrado de bloques en Linux. La versión 2 introduce `argon2id` para la derivación de claves (resistente a ataques por fuerza bruta con GPU) y `aes-xts-plain64` como modo de cifrado por defecto. Es fundamental comprender que **LUKS no cifra los datos directamente**, sino que protege una *clave maestra* almacenada en la cabecera del volumen; la frase contraseña que ingresa el usuario sirve únicamente para desbloquear dicha clave.

#### 1. Inicialización del contenedor LUKS2

``` bash
$ sudo cryptsetup luksFormat --type luks2 --label datos /dev/sda4

¡ATENCIÓN!
==========
Esto sobreescribirá los datos en /dev/sda4 de forma irrevocable.

¿Está seguro? (Teclee 'yes' en mayúsculas): YES
Introduzca la frase contraseña de /dev/sda4: 
Verifique la frase contraseña: 
```

> **Advertencia de seguridad:**\
> Guarde y recuerde muy bien la **frase contraseña**, ya que si la pierde, será **imposible** abrir el volumen encriptado. LUKS no posee mecanismos de recuperación ni backdoors criptográficos.

#### 2. Apertura del volumen y creación del mapeador

Un dispositivo LUKS cerrado es ilegible para el sistema. `luksOpen` descifra la clave maestra en memoria RAM y expone un dispositivo virtual en `/dev/mapper/`. Este mapeador actúa como un filtro transparente: todo dato escrito se cifra antes de alcanzar el disco físico; todo dato leído se descifra antes de llegar a la aplicación.

``` bash
$ sudo cryptsetup luksOpen /dev/sda4 datos
Introduzca la frase contraseña de /dev/sda4: 
$ lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINT | grep -E "sda4|datos"
└─sda4                                        673.9G crypto_LUKS datos  
  └─datos                                     673.9G                    
```

#### 3. Formateo del sistema de archivos `ext4`

Se formatea `/dev/mapper/datos`, **no** `/dev/sda4`, con un sistema de archivos `ext4` por su estabilidad, bajo consumo de memoria y compatibilidad probada con operaciones `discard` (TRIM) en el disco de estado sólido (SSD).

``` bash
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

> **Nota de preservación:**\
> Antes de continuar, se recomienda realizar un respaldo de la cabecera LUKS para garantizar la recuperación ante fallos de escritura o ante reinstalaciones futuras:\
> `sudo cryptsetup luksHeaderBackup /dev/sda4 --header-backup-file ~/luks-header-datos.img`

Con el volumen cifrado y formateado, el siguiente paso consiste en configurar su montaje automático durante el arranque del sistema, para lo cual es necesario registrar su identificación en los archivos de configuración de `systemd`.

#### V. Fase 3: Configuración de persistencia y montaje automático

Para que el volumen sobreviva a reinicios, `systemd` debe conocer dos parámetros fundamentales: *cómo abrir el cifrado* y *dónde montarlo*. Esta información se registra en `/etc/crypttab` y `/etc/fstab`, respectivamente, y es interpretada por generadores automáticos de unidades de servicio durante la fase de prearranque.

#### 1. Obtención del identificador único (UUID)

Antes de registrar el volumen cifrado en los archivos de configuración del sistema, es indispensable obtener su UUID. Este identificador es generado aleatoriamente durante la inicialización del contenedor LUKS y permanece inmutable a lo largo de la vida útil del dispositivo, incluso si el kernel reordena la nomenclatura de los bloques (ej. `/dev/sda4` → `/dev/sdb1` tras conectar un disco externo).

``` bash
$ sudo blkid /dev/sda4
/dev/sda4: UUID="54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7" TYPE="crypto_LUKS" LABEL="datos" PARTUUID="c607c2f0-fc0d-43e4-960d-ca538edfff6c"
```

El valor que figura tras `UUID=` corresponde al identificador de la partición física cifrada. Debe copiarse íntegramente para ser insertado en `/etc/crypttab`. `systemd-cryptsetup` utiliza este UUID como referencia absoluta durante la fase de `initramfs`, garantizando que el mapeador se asocie correctamente al dispositivo incluso en entornos con múltiples discos o controladores de almacenamiento de distinta arquitectura.

#### 2. Edición de archivos de configuración con `vi`

**`/etc/crypttab`** (desencriptación en arranque):

``` bash
$ sudo vi /etc/crypttab
```

*(Presione `i` para entrar en modo inserción. Añada la línea al final. Presione `Esc`, luego `:wq` y `Enter` para guardar y salir.)*

```         
datos UUID=54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7 none luks,discard
```

| Campo          | Significado técnico                                       |
|--------------------|----------------------------------------------------|
| `datos`        | Nombre del mapeador en `/dev/mapper/`                     |
| `UUID=...`     | Identificador inmutable de la partición física            |
| `none`         | Solicita la frase interactivamente mediante Plymouth      |
| `luks,discard` | Activa LUKS2 y permite que TRIM atraviese la capa cifrada |

**`/etc/fstab`** (montaje del sistema de archivos):

``` bash
$ sudo vi /etc/fstab
```

*(Mismo procedimiento de edición que en el paso anterior.)*

```         
/dev/mapper/datos /backup ext4 defaults,noatime 0 2
```

| Campo | Significado técnico |
|--------------------|----------------------------------------------------|
| `/dev/mapper/datos` | Referencia al dispositivo desencriptado |
| `/backup` | Punto de montaje en la jerarquía del sistema de archivos raíz |
| `defaults,noatime` | Opciones estándar + omisión de timestamps de lectura para optimizar SSD |
| `0 2` | Omite `dump`; habilita verificación `fsck` tras el montaje de la raíz |

#### 3. Regeneración de `initramfs` y aplicación de cambios

El kernel carga un sistema mínimo en memoria (`initramfs`) antes de montar la partición raíz. `dracut` reconstruye esta imagen para incluir `systemd-cryptsetup` y leer `crypttab`. Posteriormente, `systemd` debe recargar su configuración para reconocer `fstab`.

``` bash
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

> **Interpretación:**\
> Tras recargar `systemd`, el volumen se montó correctamente. La asignación de propiedad garantiza que el usuario `alumno` opere con permisos de lectura/escritura sin requerir privilegios de superusuario.

##### Comentarios críticos para recuperación tras reinstalación de Fedora

Una reinstalación limpia del sistema operativo **preserva intactos los datos cifrados** en `/dev/sda4`, pero sobrescribe `/etc/crypttab`, `/etc/fstab` y la imagen `initramfs`. Para restablecer el acceso sin perder el respaldo, siga esta secuencia una vez finalizada la nueva instalación:

1.  Verifique el UUID original: `sudo blkid /dev/sda4`
2.  Recree las entradas en `/etc/crypttab` y `/etc/fstab` exactamente como se muestra arriba.
3.  Regenere la imagen de arranque: `sudo dracut -f`
4.  Restaure los contextos de seguridad y la propiedad: `sudo restorecon -Rv /backup && sudo chown -R alumno:alumno /backup`
5.  Reinicie o ejecute `sudo mount -a` para validar el montaje automático.

Esta garantía de recuperación está intrínsecamente ligada a la preservación de la frase de contraseña y del UUID del volumen. Mientras ambos elementos estén disponibles, la reconstrucción de la configuración de montaje es un proceso lineal y reversible.

#### VI. Fase 4: Respaldo de la cabecera LUKS para recuperación ante fallos

Hasta este punto, el volumen cifrado está operativo y montado automáticamente. Sin embargo, la arquitectura LUKS introduce un punto de vulnerabilidad crítico: **la cabecera del volumen**. Este bloque de metadatos, ubicado en los primeros 16 MiB del dispositivo, contiene la clave maestra, la configuración de algoritmos y las ranuras de clave. Su corrupción —por fallos de escritura, cortes de energía durante operaciones de particionamiento o sobrescritura accidental— implica la pérdida irreversible de los datos, independientemente de que el bloque de datos cifrados permanezca intacto.

#### 1. Generación del respaldo de cabecera

El comando `luksHeaderBackup` crea una copia binaria exacta de la cabecera LUKS, preservando todas las claves y configuraciones. Este respaldo debe generarse inmediatamente después de `luksFormat` y actualizarse cada vez que se modifique la configuración de claves.

``` bash
$ sudo cryptsetup luksHeaderBackup /dev/sda4 --header-backup-file ~/luks-header-datos.img
$ ls -lh ~/luks-header-datos.img
-rw------- 1 alumno alumno 16M may 29 14:20 /home/alumno/luks-header-datos.img
```

> **Nota de seguridad:**\
> El archivo `luks-header-datos.img` contiene información criptográfica sensible. Guárdelo en un medio físico distinto al disco cifrado (USB externo, servidor institucional o almacenamiento en la nube cifrado) y proteja su integridad con un checksum:\
> `sha256sum ~/luks-header-datos.img > ~/luks-header-datos.img.sha256`

#### 2. Procedimiento de restauración de la cabecera

> **Advertencia crítica:**\
> La operación `luksHeaderRestore` **sobrescribe la cabecera del dispositivo destino**. Si se ejecuta sobre un dispositivo incorrecto o con un archivo de respaldo desactualizado, **los datos serán irrecuperables**. **Verifique tres veces el dispositivo y el archivo antes de proceder.**

##### 2.1. Verificación previa a la restauración

``` bash
### 1. Verificar que el archivo de respaldo existe y tiene tamaño razonable (~16 MiB para LUKS2)
$ ls -lh ~/luks-header-datos.img
-rw------- 1 alumno alumno 16M may 29 14:20 /home/alumno/luks-header-datos.img

### 2. Confirmar que el dispositivo destino es el esperado
$ sudo blkid /dev/sda4
/dev/sda4: PARTUUID="c607c2f0-fc0d-43e4-960d-ca538edfff6c"

### 3. (Opcional) Inspeccionar metadatos del respaldo sin restaurar
$ sudo cryptsetup luksDump ~/luks-header-datos.img | head -20
```

##### 2.2. Verificación de integridad y ejecución de la restauración

Dado que la operación de restauración modifica irreversiblemente los metadatos del dispositivo, el procedimiento debe dividirse en dos etapas claramente diferenciadas: una **verificación no destructiva** del archivo de respaldo y la **ejecución controlada** de la sobrescritura.

###### Paso A: Validación del respaldo con `luksDump` (solo lectura)
Antes de interactuar con el dispositivo físico, es imperativo confirmar que `~/luks-header-datos.img` contiene una cabecera LUKS2 válida, estructurada y coherente con los parámetros originales. El comando `luksDump` inspecciona el archivo en modo de solo lectura, sin alterar ni el respaldo ni el disco.

```bash
$ sudo cryptsetup luksDump ~/luks-header-datos.img | head -20
LUKS header information
Version:       	2
Epoch:         	3
Metadata area: 	16384 [0x0000000000004000]
UUID:          	54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7
Label:         	datos
Cipher:        	aes
Cipher mode:   	xts-plain64
Hash:          	sha256
Integrity:     	hmac-sha256
Payload offset:	32768
MK bits:       	512
MK digest:     	a1 b2 c3 d4 e5 f6 07 18 ... (valores hexadecimales aleatorios)
MK salt:       	fa 9b 8c 7d 6e 5f 4a 3b ... (valores hexadecimales aleatorios)

Keyslots:
0: luks2
	Key:        512 bits
	Priority:   normal
	Cipher:     aes-xts-plain64
	PBKDF:      argon2id
```

**Interpretación técnica de los campos:**

| Línea/Campo | Significado técnico |
|-------------|-------------------|
| `Version: 2` | Confirma el formato LUKS2, que estructura los metadatos en JSON, tolera fallos parciales y soporta `argon2id`. |
| `Epoch: 3` | Contador de revisiones de la cabecera. Incrementa automáticamente ante cada modificación de clave, adición de ranura o actualización de tokens. |
| `Metadata area: 16384` | Área de metadatos de 16 KiB, alineada a bloques de 512 bytes según la especificación LUKS2. |
| `UUID: ...` | Identificador criptográfico único. Debe coincidir exactamente con el registrado en `/etc/crypttab` y con `blkid /dev/sda4`. |
| `Label: datos` | Etiqueta legible asignada durante `luksFormat --label datos`. |
| `Cipher: aes` / `Cipher mode: xts-plain64` | Algoritmo y modo de operación por defecto en LUKS2. `xts-plain64` está optimizado para cifrado de bloques secuenciales y evita patrones reconocibles en datos repetidos. |
| `Hash: sha256` / `Integrity: hmac-sha256` | Función hash para derivación de claves y verificación de autenticidad de la cabecera. |
| `Payload offset: 32768` | Desplazamiento en sectores de 512 B donde inicia el bloque de datos cifrados. Equivale a 16 MiB, dimensionado para alojar cabecera primaria, cabecera secundaria y metadatos JSON. |
| `MK bits: 512` | Longitud de la clave maestra (256 bits para cifrado + 256 bits para autenticación en modo XTS). |
| `Keyslots: 0: luks2` | Primera ranura de clave activa. Derivada mediante `argon2id`, parámetro resistente a aceleración por hardware especializado (GPU/ASIC). |

> **Nota de validación:** Si la salida inicia con `LUKS header information` y reporta `Version: 2`, el respaldo es estructuralmente íntegro. Si el archivo estuviera corrupto o no correspondiera a un contenedor LUKS, `cryptsetup` retornaría: `Device ~/luks-header-datos.img is not a valid LUKS device`.

###### Paso B: Ejecución de la restauración (`luksHeaderRestore`)
Una vez validado el respaldo, se procede a sobrescribir la cabecera del dispositivo físico. Este comando reemplaza los primeros ~16 MiB de `/dev/sda4` con el contenido binario del archivo de respaldo.

```bash
$ sudo cryptsetup luksHeaderRestore /dev/sda4 --header-backup-file ~/luks-header-datos.img
```

**Comportamiento operativo:**
- **Sin confirmación interactiva:** Por diseño, `luksHeaderRestore` no solicita validación adicional. La ejecución es inmediata y destructiva sobre la cabecera destino.
- **Salida silenciosa en éxito:** El comando no retorna mensajes en terminal si la operación finaliza correctamente. La ausencia de errores (`stderr`) constituye la confirmación de éxito.
- **Irreversibilidad:** Tras la ejecución, la cabecera original del dispositivo queda sobrescrita. Si el archivo de respaldo corresponde a una configuración de claves distinta a la esperada, el volumen no podrá abrirse con la frase actual hasta que se restaure un respaldo coherente.

###### Paso C: Validación post-restauración
Para certificar que la cabecera fue aplicada correctamente y que el volumen es nuevamente accesible:

```bash
### 1. Verificar que el dispositivo reporta la cabecera restaurada
$ sudo cryptsetup luksDump /dev/sda4 | grep -E "Version|Label|UUID"
Version:       	2
Label:         	datos
UUID:          	54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7

### 2. Validar apertura con la frase de contraseña original
$ sudo cryptsetup luksOpen /dev/sda4 datos
Introduzca la frase contraseña de /dev/sda4: 

### 3. Confirmar montaje y disponibilidad de datos
$ sudo mount /dev/mapper/datos /backup
$ ls /backup/respaldos_home/
20260529
```

Esta secuencia garantiza que la recuperación de metadatos criptográficos se ejecute con validación explícita, minimizando el riesgo de pérdida de datos por sobrescritura inadvertida o uso de respaldos desactualizados.

##### 2.3. Validación post-restauración

``` bash
### 1. Verificar que la cabecera fue restaurada correctamente
$ sudo cryptsetup luksDump /dev/sda4 | grep -E "Version|Label"
Version:        2
Label:          datos

### 2. Intentar abrir el volumen con la frase de contraseña original
$ sudo cryptsetup luksOpen /dev/sda4 datos
Introduzca la frase contraseña de /dev/sda4: 

### 3. Confirmar montaje y acceso a datos
$ sudo mount /dev/mapper/datos /backup
$ ls /backup/respaldos_home/
20260529
```

#### 3. Consideraciones técnicas y buenas prácticas

| Aspecto | Recomendación |
|----------------------------|-------------------------------------------|
| **Almacenamiento del respaldo** | Guarde `luks-header-datos.img` en un medio físico distinto al disco cifrado. Su pérdida junto con el dispositivo original anula su utilidad. |
| **Actualización del respaldo** | Si cambia la frase de contraseña o añade ranuras de clave con `cryptsetup luksAddKey`, regenere el respaldo. Un respaldo desactualizado no contendrá las nuevas claves. |
| **Compatibilidad de versiones** | Restaure siempre el respaldo en el mismo dispositivo o en uno de capacidad idéntica o mayor. La restauración en un disco más pequeño puede corromper la estructura de particiones. |
| **Integridad del archivo** | Calcule y registre un checksum del respaldo al crearlo: `sha256sum ~/luks-header-datos.img > luks-header-datos.img.sha256`. Verifique antes de restaurar: `sha256sum -c luks-header-datos.img.sha256`. |
| **Auditoría sin riesgo** | Use `luksDump` sobre el archivo de respaldo para verificar algoritmos y ranuras de clave sin exponer el dispositivo activo: `sudo cryptsetup luksDump ~/luks-header-datos.img`. |

#### 4. Escenarios de uso típicos

| Escenario | Procedimiento recomendado |
|----------------------|--------------------------------------------------|
| **Corrupción de cabecera por fallo de escritura** | Restaurar directamente con `luksHeaderRestore` tras verificar integridad del archivo de respaldo. |
| **Reinstalación accidental de `/dev/sda4`** | Si la partición fue formateada pero no sobrescrita completamente, intente restaurar la cabecera; si los datos fueron sobrescritos, la recuperación no es posible. |
| **Migración a nuevo disco** | Copie el bloque de datos completo con `dd` o `ddrescue`, luego restaure la cabecera LUKS en el destino para preservar las claves originales. |
| **Auditoría de seguridad** | Use `luksDump` sobre el respaldo para verificar algoritmos, ranuras de clave y etiquetas sin exponer el dispositivo activo. |

> **Resumen operativo:**\
> 1. Cree el respaldo inmediatamente después de `luksFormat`.\
> 2. Almacénelo en un medio seguro y externo al disco cifrado.\
> 3. En caso de corrupción, restaure con `luksHeaderRestore` tras verificar dispositivo y archivo.\
> 4. Valide el acceso con `luksOpen` y `mount` antes de declarar la recuperación exitosa.

Con el volumen cifrado, configurado para montaje automático y protegido mediante respaldo de cabecera, se procede a ejecutar el respaldo propiamente dicho, empleando herramientas que garanticen la integridad de los datos y la resiliencia ante interrupciones.

#### VII. Fase 5: Ejecución resiliente del respaldo con `tmux` y `rsync`

Un respaldo de \~150 GiB puede extenderse entre 30 y 90 minutos. Si la terminal se cierra, la conexión SSH se interrumpe o el sistema suspende la sesión, `rsync` se terminará abruptamente, corrompiendo potencialmente la copia. `tmux` (terminal multiplexer) ejecuta procesos en sesiones virtuales independientes de la terminal física, permitiendo desconectarse y reconectarse sin perder el progreso ni el contexto de ejecución.

#### 1. Sesión resiliente con `tmux`

``` bash
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

> 🔹 **Desvinculación segura sin detener el proceso:**\
> Presione `Ctrl+B`, suelte ambas teclas y presione `D`. La terminal se liberará, pero `rsync` continuará ejecutándose en segundo plano dentro de `tmux`.
>
> 🔹 **Reconexión o recuperación de la sesión:**
>
> ``` bash
> $ tmux attach -t respaldo
> ```
>
> El usuario recupera la consola exactamente en el punto donde quedó, sin interrupciones ni pérdida de contexto.

#### 2. Salida final del proceso de respaldo

```         
sent 166,565,358,492 bytes  received 597,485 bytes  39,672,729.78 bytes/sec
total size is 166,522,513,580  speedup is 1.00
```

> **Interpretación:**\
> La velocidad promedio (\~39.7 MB/s) es coherente con la sobrecarga mínima introducida por el cifrado LUKS2. `tmux` garantizó la ejecución ininterrumpida, demostrando su valor en operaciones de mantenimiento crítico.

#### 3. Salida de la sesión resiliente `tmux`

``` bash
$ exit
```

#### VIII. Fase 6: Validación integral y prueba de ciclo completo

La validación definitiva consiste en cerrar el volumen, reiniciar el equipo y verificar que el sistema solicita la frase de cifrado durante el arranque, desencripta el medio y lo monta automáticamente.

#### 1. Preparación y reinicio

``` bash
$ sudo umount /backup
$ sudo cryptsetup luksClose datos
$ sudo reboot
```

#### 2. Validación posterior al reinicio

``` bash
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

> **Conclusión de la prueba:**\
> El servicio `systemd-cryptsetup` desencriptó el volumen en fase temprana de arranque, `fstab` lo montó automáticamente y los datos son accesibles. La configuración es persistente, funcional y lista para producción.

#### IX. Registro técnico consolidado

| Parámetro                | Valor configurado                                |
|---------------------------|---------------------------------------------|
| **Dispositivo físico**   | `/dev/sda4` (673.9 GiB)                          |
| **Etiqueta LUKS**        | `datos`                                          |
| **UUID LUKS**            | `54acac45-d8c9-4b6d-8cd3-9f7cefb39ca7`           |
| **Nombre de mapeo**      | `datos` → `/dev/mapper/datos`                    |
| **Sistema de archivos**  | `ext4` (etiqueta: `datos_vol`)                   |
| **Punto de montaje**     | `/backup`                                        |
| **Opciones `crypttab`**  | `none luks,discard`                              |
| **Opciones `fstab`**     | `defaults,noatime 0 2`                           |
| **Datos respaldados**    | `~156 GiB` en `/backup/respaldos_home/20260529/` |
| **Respaldo de cabecera** | `~/luks-header-datos.img` (16 MiB)               |
| **Estado operativo**     | alidado en ciclo completo de arranque        |

#### X. Conclusiones

Este laboratorio demuestra que la seguridad de datos y la automatización del arranque no requieren infraestructura compleja ni soluciones propietarias. Mediante herramientas nativas de Fedora (`cryptsetup`, `systemd`, `dracut`, `rsync`), es posible construir un flujo de respaldo cifrado, persistente y resiliente a interrupciones de sesión. La integración de `tmux` ilustra una práctica administrativa fundamental: desacoplar la ejecución de procesos críticos de la estabilidad de la terminal cliente, garantizando la finalización exitosa de operaciones de larga duración sin supervisión constante.

El esquema resultante cumple con los principios de confidencialidad (cifrado LUKS2 con `argon2id`), integridad (preservación de metadatos, ACLs y contextos SELinux mediante `rsync -aAXHv`) y disponibilidad (montaje automático gestionado por `systemd` y validación cíclica). La incorporación del respaldo de cabecera LUKS añade una capa esencial de resiliencia ante fallos de metadatos, completando un ciclo de protección integral.

Su implementación en entornos académicos e institucionales no solo protege la información sensible, sino que establece una base reproducible para la adopción de prácticas de ciberseguridad alineadas con estándares internacionales. Se recomienda complementar este flujo con políticas de retención automatizada, verificación de integridad mediante checksums y almacenamiento seguro de la frase de contraseña y la cabecera LUKS en medios externos seguros.

#### XI. Referencias

cryptsetup project. (2025). *LUKS2 on-disk format specification* (Version 1.1.4). GitLab. <https://gitlab.com/cryptsetup/LUKS2-docs>

cryptsetup project. (2026). *cryptsetup: Userspace setup tool for dm-crypt/LUKS encrypted block devices*. GitLab. <https://gitlab.com/cryptsetup/cryptsetup>

Fedora Project. (2026). *Disk encryption user guide*. Fedora Documentation. <https://docs.fedoraproject.org/en-US/quick-docs/encrypting-drives-using-LUKS/>

Fedora Project. (2026). *Dracut*. Fedora Wiki. <https://fedoraproject.org/wiki/Dracut>

Fruhwirth, C. (2004). *Linux Unified Key Setup (LUKS) disk encryption specification*. <https://gitlab.com/cryptsetup/cryptsetup/wikis/home>

Samba Team. (2026). *rsync(1) manual page*. <https://download.samba.org/pub/rsync/rsync.1>

systemd project. (2026). *crypttab(5)*. Freedesktop.org. <https://www.freedesktop.org/software/systemd/man/crypttab.html>

systemd project. (2026). *systemd-cryptsetup-generator(8)*. Freedesktop.org. <https://www.freedesktop.org/software/systemd/man/systemd-cryptsetup-generator.html>

systemd project. (2026). *systemd-fstab-generator(8)*. Freedesktop.org. <https://www.freedesktop.org/software/systemd/man/systemd-fstab-generator.html>

tmux project. (2026). *tmux: Terminal multiplexer*. GitHub Wiki. <https://github.com/tmux/tmux/wiki>

Ziegler, H. (Ed.). (2026). *dracut(8): Low-level tool for generating an initramfs/initrd image*. man7.org Linux man-pages. <https://man7.org/linux/man-pages/man8/dracut.8.html>
