# Instalación y Comunicaciones - Tasca 5 (ASIX ThinkLab)

| Datos de Configuración | Detalle |
| :--- | :--- |
| **Alumno** | Darío Martiáñez Pérez |
| **Número de PC** | PC-26 |
| **Red Interna (LAN)** | `192.168.26.0/24` |
| **Módulos** | ISO / FDM |

---

## 1. Introducción y Objetivos de la Tasca 5

El presente documento recoge la implantación práctica, configuración y verificación de los sistemas operativos y servicios esenciales que componen el entorno de pruebas individualizado del departamento de informática, adaptado al puesto de trabajo **PC-26**.

Siguiendo los criterios de evaluación de los módulos de **ISO** e **FDM**, esta fase abarca el despliegue de los servidores de comunicaciones, almacenamiento NAS, clúster de virtualización de alta disponibilidad, centralización de dominios y servicios de infraestructura, garantizando el aislamiento de la red interna, el enrutamiento efectivo hacia el exterior y la consolidación de un sistema de copias de seguridad centralizado. Asimismo, se incorporan las respuestas a las cuestiones teóricas e investigación requeridas en el enunciado oficial de la tarea.

---

## 2. Comprobación de Origen e Integridad de las ISOs

Para garantizar que los medios de instalación empleados en el despliegue del entorno no han sido alterados y coinciden de manera íntegra con las fuentes oficiales de los fabricantes, se ha llevado a cabo una validación automatizada mediante huellas criptográficas `sha256sum`. 

Se han contrastado las sumas locales almacenadas en el fichero `hash.txt` frente a las referencias oficiales registradas en `hash_oficial.txt`, utilizando la utilidad de comparación `diff` para certificar la coincidencia exacta de los ficheros operativos.

### A. Metodología de Validación y Justificación
La integridad criptográfica asegura que las imágenes del sistema operativo descargadas no contienen modificaciones maliciosas ni errores de transmisión. El proceso automatizado verifica cada archivo ISO comparando su hash SHA-256 local con el proporcionado por el desarrollador:
1. Generación de sumas locales: `sha256sum * > hash.txt`
2. Contrastado estricto mediante la ejecución del comando: `diff -u hash_oficial.txt hash.txt`
3. Un resultado sin salida en el comando `diff` certifica matemáticamente la coincidencia absoluta.

### B. Evidencia de Validación de Hashes
- **Comprobación mediante `diff` y volcado de los ficheros locales y oficiales:**
  > ![Comprobación Hashes ISOs](../evidencias/t5-comprobacion_isos.png)

---

## 3. Servidores de Comunicaciones (Gestión de Incidencias y Trazabilidad)

Para cumplir con los requisitos de diseño y resiliencia del sistema, se han implementado y probado sucesivamente **tres alternativas de servidores de comunicaciones y cortafuegos** (`MikroTik`, `OPNsense` y `deb-comm`). En todas ellas se han configurado los usuarios de administración específicos, las interfaces de red duales (WAN conectada a la red externa por DHCP y LAN estática en `192.168.26.1/24`), y las reglas de filtrado y enrutamiento necesarias para proporcionar acceso transparente a Internet a toda la red interna.

### A. Identificación y Respuesta ante Incidencias de Red
Con el fin de cumplir con la detección de configuraciones críticas y tiempos de respuesta ante eventos no deseados:
- Se implementó un bloqueo explícito hacia la IP externa `1.1.1.1` (Cloudflare) con registro obligatorio de paquetes (`log=yes` / `log prefix`).
- **Interpretación del tiempo de respuesta e incidente:** Ante la suplantación o intento de salida por rutas no autorizadas, los firewalls generan de manera inmediata trazas de auditoría (`/log print`, *Live View*, y `dmesg`). Esto permite al administrador identificar el origen exacto (`192.168.26.109`) y cortar el tráfico de forma preventiva en menos de un segundo.

### B. Resumen de Configuración de Gateways
| Servidor de Comunicaciones | Interfaz WAN (Externa) | Interfaz LAN (Interna) | Usuario de Administración | Estado de Verificación |
| :--- | :--- | :--- | :--- | :--- |
| **MikroTik RouterOS** | `ether2` (`10.0.3.15` - DHCP) | `ether1` (`192.168.26.1/24`) | `admin` / `dario` | Verificado (Concurrencia única) |
| **OPNsense** | `wan` (`10.0.3.15` - DHCP) | `lan` (`192.168.26.1/24`) | `dario` / `root` | Verificado (Concurrencia única) |
| **Debian con nftables (`deb-comm`)** | `enp0s8` (`10.0.3.15` - DHCP) | `enp0s3` (`192.168.26.1/24`) | `batoi` y `dario` (Sudoers) | Verificado (Concurrencia única) |

### C. Evidencias de Ejecución y Configuración

#### 1. MikroTik RouterOS
- **Inicio de sesión con el usuario de administración creado:**
  > ![Sesión Usuario MikroTik](../evidencias/t5-dario_microtik.png)

- **Generación y transferencia de copia de seguridad (`scp`):**
  > ![Generar backup MikroTik](../evidencias/t5-backup_mikrotik_generar.png)
  > ![SCP backup MikroTik](../evidencias/t5-backup_mikrotik_scp.png)
  > ![Exportar copia MikroTik](../evidencias/t5-exportar_copia_microtik.png)

- **Filtrado de tráfico (Bloqueo a `1.1.1.1`) y verificación de registros (`/log print`):**
  *(Se muestra la regla `drop` en la cadena `forward` con prefijo de registro `BLOQUEADO_1.1.1.1:`, el bloqueo efectivo del ping hacia `1.1.1.1` frente al tráfico permitido hacia `8.8.8.8` desde `deb-cli`, y las trazas capturadas en el log del router).*
  > ![Conexiones y Bloqueo MikroTik](../evidencias/t5-mikrotik_conexiones.png)

#### 2. OPNsense
- **Configuración del usuario de administración (`dario`):**
  > ![Usuario Admin OPNsense](../evidencias/t5-dario_opnsense.png)

- **Descarga del respaldo de configuración desde el panel web al cliente:**
  > ![Descarga Backup OPNsense](../evidencias/t5-baja_bak_opnsense.png)

- **Filtrado de tráfico (Bloqueo a `1.1.1.1`) y supervisión en tiempo real (*Live View*):**
  *(Se muestra en el panel Live View de OPNsense el registro en rojo de los paquetes ICMP bloqueados por la regla `Bloqueo IP 1.1.1.1` en la interfaz LAN, contrastado con la pérdida del 100% de paquetes en el terminal de `deb-cli`).*
  > ![Conexiones y Bloqueo OPNsense](../evidencias/t5-opnsense_conexiones.png)

#### 3. Servidor de Comunicaciones Debian (`deb-comm`)
- **Creación del usuario de administración (`dario`) e inclusión en el grupo `sudo`:**
  > ![Usuario Admin deb-comm](../evidencias/t5-dario_debcomm.png)

- **Generación de respaldo mediante script (`.tar.gz`) y transferencia por `scp` a `deb-cli`:**
  *(Se muestra el contenido del script de copia de seguridad, su ejecución, el listado del fichero comprimido resultante, el cierre de la sesión SSH en `deb-comm` y la descarga segura mediante `scp` desde el cliente `deb-cli`).*
  > ![Script y SCP Backup deb-comm](../evidencias/t5-exportar_copia_debcomm.png)
  > ![Backup debcomm](../evidencias/t5-backup_debcomm.png)

- **Filtrado con `nftables` (Bloqueo a `1.1.1.1`) y trazas del núcleo (`dmesg`):**
  *(Se observa la monitorización del kernel mediante `dmesg` registrando el prefijo `NFT_BLOQUEO_1.1.1.1` en los paquetes reenviados desde `enp0s3` hacia `enp0s8`, junto con la confirmación del corte de conexión en el cliente `deb-cli`).*
  > ![Conexiones y Bloqueo deb-comm](../evidencias/t5-deb-comm_conexiones.png)

---

## 4. Servidor de Almacenamiento y NAS (`store-srvv`) - Política de Respaldo

Como pilar fundamental para la retención de datos y la política de resiliencia de la organización, se ha implementado el servidor de almacenamiento basado en **TrueNAS SCALE (`store-srvv`)**. 

Se han configurado los parámetros de red estáticos, los accesos mediante usuarios de administración seguros, y se ha añadido un **disco virtual adicional** en el hipervisor dedicado a albergar el pool de almacenamiento y el dataset estructurado para la recepción de copias de seguridad y respaldos de imágenes de los sistemas críticos.

### A. Justificación Técnica: Qué, Cómo y Dónde se Almacena
* **¿Qué se copia?** Archivos de configuración de red, scripts de automatización, bases de datos de servicios (`/etc`, `/home`, configuraciones de cortafuegos) y snapshots de máquinas virtuales. Se excluyen deliberadamente directorios temporales (`/tmp`), cachés de paquetes (`/var/cache/apt`) e historiales de sesión para optimizar el espacio y evitar corrupciones.
* **¿Cómo se realiza?** Mediante compresión estructurada en paquetes `.tar.gz` con marcas temporales y transferencia segura por protocolo cifrado (`scp` / API).
* **¿Dónde se aloja?** En un dataset ZFS dedicado en el servidor NAS de la red (`192.168.26.5:/mnt/pool_backups/backups`), garantizando redundancia y aislamiento frente a fallos en los nodos de producción.

### B. Parámetros Técnicos de Almacenamiento
- **Dirección IP LAN:** `192.168.26.5/24`
- **Sistema de Ficheros:** ZFS (Pool dedicado a respaldos)
- **Servicios Habilitados:** SMB / SSH / API de Gestión

### C. Evidencias del Servidor NAS
- **Configuración del disco virtual adicional en VirtualBox:**
  > ![Disco store-srvv](../evidencias/t5-disco_storesrvv_vbox.png)

- **Parámetros de red y conectividad IP:**
  > ![Red store-srvv](../evidencias/t5-network_storesrvv.png)
  > ![Acceso SSH store-srvv](../evidencias/t5-ssh_storesrvv.png)

- **Creación de usuarios y pools de almacenamiento ZFS:**
  > ![Usuario store-srvv](../evidencias/t5-user_storesrvv.png)
  > ![Dataset store-srvv](../evidencias/t5-dataset_storesrvv.png)
  > ![Pool Backups store-srvv](../evidencias/t5-pool_backups_storesrvv.png)
  > ![Config store-srvv 1](../evidencias/t5-dario_storesrvv_1.png)
  > ![Config store-srvv 2](../evidencias/t5-dario_storesrvv_2.png)

- **Verificación de la integridad de los datasets de respaldo:**
  > ![Verificación Copias NAS](../evidencias/t5-verificacion_backups_storesrvv.png)

---

## 5. Clúster de Virtualización (`Proxmox VE - PX`)

Se ha desplegado un clúster de virtualización de alta disponibilidad compuesto por **tres nodos** (`N1-PX`, `N2-PX`, `N3-PX`), interconectados en la red interna de la organización y vinculados al almacenamiento centralizado en TrueNAS. 

Se han llevado a cabo pruebas de creación de máquinas virtuales y contenedores, así como la migración en caliente entre nodos para comprobar el rendimiento y la eficiencia del clúster bajo los recursos dimensionados.

### A. Dimensión de Nodos del Clúster
| Nodo Proxmox | Dirección IP | vCPU | RAM Asignada | Almacenamiento | Rol en el Clúster |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **`N1-PX`** | `192.168.26.2/24` | 2 | 4048 MB | 32,00 GB | Nodo 1 (Coordinador) |
| **`N2-PX`** | `192.168.26.3/24` | 2 | 4048 MB | 32,00 GB | Nodo 2 (Cómputo) |
| **`N3-PX`** | `192.168.26.4/24` | 2 | 4048 MB | 32,00 GB | Nodo 3 (Cómputo / Alta Disponibilidad) |

### B. Análisis Técnico e Investigación Requerida

#### 1. Ventajas e Inconvenientes de la Virtualización de Servidores
- **Ventajas:**
  - **Ahorro de costes y consolidación de hardware:** Permite ejecutar múltiples sistemas operativos y servicios sobre una única máquina física, reduciendo costes de adquisición y consumo eléctrico.
  - **Flexibilidad y Aprovisionamiento Rápido:** Capacidad de crear, clonar y desplegar nuevos servidores en pocos minutos mediante plantillas.
  - **Aislamiento:** Las fallas o caídas en una máquina virtual no afectan al resto de servicios del sistema.
  - **Portabilidad:** Los servidores virtuales son archivos independientes que se pueden mover fácilmente entre diferentes hipervisores.
- **Inconvenientes:**
  - **Sobrecarga de rendimiento (*Overhead*):** El hipervisor consume recursos adicionales de CPU y memoria para gestionar la capa de virtualización.
  - **Punto único de fallo físico:** Si el nodo anfitrión físico (*host*) sufre una avería catastrófica sin redundancia, todas las máquinas virtuales alojadas en él quedan inaccesibles simultáneamente.
  - **Complejidad de gestión:** Requiere conocimientos avanzados de redes virtuales, almacenamiento compartido y monitorización.

#### 2. Diferencia entre Contenedores y Máquinas Virtuales (¿Cuándo utilizar cada uno?)
- **Máquinas Virtuales (VMs):** Emulan un ordenador completo incluyendo hardware virtual, controladores y un sistema operativo invitado (*Guest OS*) completo con su propio núcleo (*kernel*). Consumen muchos más recursos (RAM y disco).
- **Contenedores (LXC / Docker):** Comparten el núcleo del sistema operativo anfitrión y virtualizan únicamente la capa de usuario. Son extremadamente ligeros y arrancan en segundos.
- **¿Cuándo utilizar cada uno?**
  - *Usar Contenedores:* Para aislar aplicaciones web, microservicios, entornos de desarrollo o bases de datos ligeras donde se busca la máxima eficiencia de recursos y velocidad de despliegue.
  - *Usar Máquinas Virtuales:* Cuando necesitamos ejecutar sistemas operativos diferentes (por ejemplo, ejecutar Windows Server sobre un nodo Proxmox Linux), cuando se requiere un aislamiento de seguridad estricto a nivel de núcleo o cuando la aplicación depende estrechamente de controladores de hardware específicos.

#### 3. Migración en Caliente (*Live Migration*) - Tiempos, Espacio y Valoración de Mejoras
Se han realizado pruebas de migración de una máquina virtual y un contenedor entre los nodos del clúster:
- **Contenedor LXC:** 
  - *Tamaño en disco:* $\approx 1,2\text{ GB}$.
  - *Tiempo de migración:* $\approx 3,4\text{ segundos}$ en red interna de $1\text{ Gbps}$.
  - *Interpretación:* Al no virtualizar un kernel completo y poseer un volumen reducido, la sincronización de memoria y metadatos es prácticamente instantánea.
- **Máquina Virtual (VM):**
  - *Tamaño en disco:* $\approx 15,0\text{ GB}$.
  - *Tiempo de migración:* $\approx 24,1\text{ segundos}$ (con memoria activa sincronizada).
  - *Interpretación:* Requiere volcar la RAM asignada y los bloques del disco virtual del host origen al destino, incrementando proporcionalmente el tiempo.
- **Valoración de mejoras:** El proceso se puede optimizar notablemente empleando almacenamiento compartido centralizado de alta velocidad (NFS/iSCSI sobre TrueNAS con discos NVMe o interfaces de red dedicadas a 10GbE), eliminando el trasvase físico de bloques de disco en caliente.

#### 4. Copias de Seguridad del Clúster Proxmox (Qué se copia y qué no)
- **¿Qué tiene sentido guardar?** Los archivos de configuración de las máquinas virtuales y contenedores (`/etc/pve/qemu-server/` y lxc), las claves de cifrado del clúster, los metadatos y las imágenes completas de los discos de los servicios críticos.
- **¿Qué NO tiene sentido guardar?** Archivos temporales del sistema (`/tmp/`), cachés de paquetes de instalación del gestor (`/var/cache/apt/archives/`), o trazas de logs obsoletas que únicamente inflan el tamaño de la copia sin aportar valor en la recuperación ante desastres.

### C. Evidencias del Clúster Proxmox
- **Estado y validación del clúster de tres nodos:**
  > ![Clúster Proxmox](../evidencias/t5-proxmox_cluster.png)
  > ![Red N1-PX](../evidencias/t5-red_N1-PX.png)
  > ![Red N2-PX](../evidencias/t5-red_N2-PX.png)
  > ![Red N3-PX](../evidencias/t5-red_N3-PX.png)
  > ![Dario N1-PX](../evidencias/t5-dario_N1-PX.png)
  > ![Dario N2-PX](../evidencias/t5-dario_N2-PX.png)
  > ![Dario N3-PX](../evidencias/t5-dario_N3-PX.png)

- **Despliegue de servidores virtuales en los nodos:**
  > ![Servidores Proxmox](../evidencias/t5-proxmox_servidores_creados.png)

- **Prueba de migración de cargas entre nodos:**
  > ![Migración Proxmox](../evidencias/t5-proxmox_migracion.png)

- **Integración de copias y snapshots con TrueNAS:**
  > ![Backups Proxmox TrueNAS](../evidencias/t5-proxmox_backups_truenas.png)

---

## 6. Centralización de Dominio (`win-srvv` y `win-cli`)

Para estructurar la gestión administrativa centralizada de identidades, directivas y equipos dentro del departamento, se ha desplegado el servidor de directorio activo sobre **Windows Server 2025 (`win-srvv`)**, integrando de forma operativa el cliente administrativo **Windows 11 (`win-cli`)**.

### A. Configuración de Dominio
- **Dirección IP del Servidor:** `192.168.26.6/24`
- **Rol Principal:** Active Directory Domain Services (AD DS) y DNS
- **Integración de Clientes:** `win-cli` asociado correctamente al dominio corporativo con asignación DHCP estática/dinámica en el rango de la LAN.

### B. Análisis e Investigación Requerida

#### 1. ¿Qué es un Dominio y qué utilidad aporta a la empresa?
- **Definición de Dominio:** Es una agrupación lógica de equipos, usuarios, impresoras y recursos de red administrados centralizadamente desde una base de datos central (Directorio Activo) controlada por uno o varios controladores de dominio.
- **Utilidad y Ventajas para la Empresa:**
  - **Gestión centralizada de identidades:** Un usuario puede iniciar sesión en cualquier equipo de la red corporativa utilizando sus credenciales únicas.
  - **Políticas de Grupo (GPOs):** Permiten aplicar configuraciones de seguridad homogéneas, restricciones de software, despliegue automático de programas y políticas de contraseñas a todos los equipos y empleados de manera instantánea.
  - **Seguridad y Control de Acceso:** Centraliza los permisos sobre recursos compartidos y carpetas de red, evitando fugas de información.

#### 2. Copias de Seguridad en el Controlador de Dominio: ¿Qué copiar y qué no?
- **¿Qué es interesante copiar?** La base de datos de Active Directory (`NTDS.dit`), la estructura del SYSVOL (scripts y directivas de grupo), el registro del sistema y las zonas de resolución DNS.
- **¿Qué NO es interesante copiar?** Archivos temporales del perfil de usuario, registros de eventos (*event logs*) antiguos sin relevancia legal o carpetas de caché local de aplicaciones.

### C. Evidencias de Centralización
- **Configuración de red y servicios en Windows Server:**
  > ![Red Windows Server](../evidencias/t5-dario_win-srvv.png)
  > ![Backup win-srvv comandos](../evidencias/t5-backup_win-srvv_comandos.png)

- **Unión e integración del cliente Windows 11 al dominio:**
  > ![Red win-cli](../evidencias/t5-red_win-cli.png)
  > ![Cliente en Dominio](../evidencias/t5-win-cli_dominio.png)

---

## 7. Servidores de Infraestructura, Servicios y Monitorización (`srv-monn`, `deb-cli`, `debian-serveis` y `debian-dades`)

Se ha configurado el servidor de monitorización basado en entornos Linux (`srv-monn`), los servidores dedicados a servicios (`debian-serveis`) y gestión de datos (`debian-dades`) dotados de herramientas de supervisión y administración activa, junto con el despliegue del entorno cliente de pruebas y validación operativo (`deb-cli`).

### A. Justificación de Respaldos en Clientes Linux (`deb-cli`)
* **¿Qué se copia en el cliente Linux?** Se respaldan las configuraciones de red locales (`/etc/network/interfaces`), ficheros de resolución de nombres y claves públicas/privadas de acceso SSH del usuario administrativo.
* **¿Qué se descarta?** Directorios temporales del sistema y binarios del sistema operativo base, ya que al tratarse de un cliente efímero de pruebas, ante un fallo catastrófico resulta más eficiente desplegar una nueva plantilla que restaurar un volcado de binarios del sistema operativo.

### B. Evidencias de Infraestructura
- **Panel de herramientas de supervisión en `srv-monn`:**
  > ![Red srv-monn](../evidencias/t5-red_srv-monn.png)
  > ![Dario srv-monn](../evidencias/t5-dario_srv-monn.png)
  > ![Herramientas Monitorización](../evidencias/t5-herramientas_srv-monn.png)

- **Configuración de red, usuarios y herramientas en `debian-serveis`:**
  > ![Red debian-serveis](../evidencias/t5-red_debian-serveis.png)
  > ![Dario debian-serveis](../evidencias/t5-dario_debian-serveis.png)
  > ![Herramientas debian-serveis](../evidencias/t5-herramientas_debian-serveis.png)

- **Configuración de red, usuarios y herramientas en `debian-dades`:**
  > ![Red debian-dades](../evidencias/t5-red_debian-dades.png)
  > ![Dario debian-dades](../evidencias/t5-dario_debian-dades.png)
  > ![Herramientas debian-dades](../evidencias/t5-herramientas_debian-dades.png)

- **Verificación de red en el cliente Linux `deb-cli`:**
  > ![Dario deb-cli](../evidencias/t5-dario_deb-cli.png)
  > ![Red deb-cli](../evidencias/t5-red_deb-cli.png)

---

## 8. Sistema de Copias de Seguridad y Respaldo (Backups)

Con el propósito de mitigar fallos imprevistos y garantizar la recuperación ante desastres (*Disaster Recovery*), se han generado, verificado y volcado de forma sistemática los respaldos de configuración y snapshots de todos los elementos críticos del sistema hacia el servidor centralizado de almacenamiento (`store-srvv` / TrueNAS):

- **Respaldo del cliente Linux (`deb-cli`):**
  > ![Backup deb-cli](../evidencias/t5-backup_deb-cli.png)

- **Respaldo del firewall Debian (`deb-comm`):**
  > ![Backup deb-comm](../evidencias/t5-backup_debcomm.png)

- **Respaldo del dispositivo de comunicaciones MikroTik:**
  > ![Backup MikroTik](../evidencias/t5-backup_mikrotik_scp.png)

- **Respaldo del servidor de monitorización (`srv-monn`):**
  > ![Backup srv-monn](../evidencias/t5-backup_srv-monn.png)

- **Respaldo del cliente administrativo (`win-cli`):**
  > ![Backup win-cli](../evidencias/t5-backup_win-cli.png)

- **Respaldo del controlador de dominio (`win-srvv`):**
  > ![Backup win-srvv](../evidencias/t5-backup_win-srvv.png)

- **Respaldo del servidor de servicios (`debian-serveis`):**
  > ![Backup debian-serveis](../evidencias/t5-backup_debian-serveis.png)

- **Respaldo del servidor de datos (`debian-dades`):**
  > ![Backup debian-dades](../evidencias/t5-backup_debian-dades.png)

---

## 9. Ampliación (Apte+) - Automatización de Respaldos Centralizados en TrueNAS (`192.168.26.5`)

Como ampliación técnica avanzada, se ha implementado la automatización total de los respaldos de configuración para todas las máquinas de la red (Linux Debian CLI, Linux Debian Com, MikroTik, OPNsense, Windows Server y Windows Client), volcando los ficheros con el formato normalizado `backup_systemname_YYYYMMDD_HHMMSS.extension` directamente en el repositorio centralizado del servidor TrueNAS (`\\192.168.26.5\backups`).

### A. Automatización en la máquina Debian CLI (`deb-cli`)
Se ha configurado un script de compresión y volcado automático ejecutado de forma desatendida mediante el planificador nativo del sistema.
* **Script de Automatización en Debian CLI:**
  > ![Script Debian CLI](../evidencias/t5-apte_deb-cli_script.png)
* **Planificador Cron en Debian CLI:**
  > ![Cron Debian CLI](../evidencias/t5-apte_deb-cli_cron.png)

### B. Automatización en la máquina Debian Com (`deb-comm`)
Se ha implementado el script de respaldo para empaquetar y transferir las reglas de red y configuraciones del cortafuegos Debian.
* **Script de Automatización en Debian Com:**
  > ![Script Debian Com](../evidencias/t5-apte_deb-com_script.png)
* **Planificador Cron en Debian Com:**
  > ![Cron Debian Com](../evidencias/t5-apte_deb-comm_cron.png)

### C. Automatización en el dispositivo MikroTik
Se configuró un script interno que genera el backup del sistema y lo transmite de forma programada al NAS.
* **Script de Automatización en MikroTik:**
  > ![Script MikroTik](../evidencias/t5-apte_mikrotik_script.png)
* **Planificador Scheduler en MikroTik:**
  > ![Scheduler MikroTik](../evidencias/t5-apte_mikrotik_scheduler.png)

### D. Automatización en el cortafuegos OPNsense
Mediante la creación del script personalizado (`/usr/local/bin/backup_auto`) integrado en el sistema operativo base FreeBSD y vinculado al Cron nativo.
* **Script de Automatización en OPNsense:**
  > ![Script OPNsense](../evidencias/t5-apte_opnsense_script.png)
* **Planificador Cron en OPNsense:**
  > ![Cron OPNsense](../evidencias/t5-apte_opnsense_cron.png)

### E. Automatización en el servidor de monitorización (`srv-monn`)
Se ha programado una tarea para respaldar la monitorización de servicios de forma periódica.
* **Script de Automatización en Servidor Monitorización:**
  > ![Script Servidor Monitorización](../evidencias/t5-apte_srv-mon_script.png)
* **Planificador Cron en Servidor Monitorización:**
  > ![Cron Servidor Monitorización](../evidencias/t5-apte_srv-monn_cron.png)

### F. Automatización en la máquina Windows Server (`win-srvv`)
Se implementó un script en PowerShell para empaquetar la configuración crítica del directorio activo y volcarla al repositorio.
* **Script de Automatización en Windows Server:**
  > ![Script Windows Server](../evidencias/t5-apte_winsrvv_script.png)
* **Tarea Programada en Windows Server:**
  > ![Tarea Windows Server](../evidencias/t5-apte_winsrvv_tarea.png)

### G. Automatización en la máquina Windows Client (`win-cli`)
Se configuró un script automatizado para respaldar el perfil de trabajo del cliente administrativo.
* **Script de Automatización en Windows Client:**
  > ![Script Windows Client](../evidencias/t5-apte_wincli_script.png)
* **Tarea Programada en Windows Client:**
  > ![Tarea Windows Client](../evidencias/t5-apte_wincli_tarea.png)

### H. Verificación del Repositorio Centralizado en TrueNAS (`store-srvv`)
Se comprueba mediante la creación del dataset y el recurso compartido que todos los respaldos automáticos heterogéneos conviven ordenadamente en el NAS.
* **Dataset de Respaldos en TrueNAS:**
  > ![Dataset TrueNAS](../evidencias/t5-apte_truenas_dataset.png)
* **Recurso Compartido en TrueNAS:**
  > ![Recurso Compartido TrueNAS](../evidencias/t5-apte_truenas_share.png)
* **Vista General del Repositorio Consolidado:**
  > ![Repositorio Centralizado TrueNAS](../evidencias/t5-apte_nas_respositorio.png)
