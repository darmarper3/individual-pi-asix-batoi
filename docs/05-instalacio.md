# Tarea 5: Instalación y Comunicaciones

## 1. Introducción y Objetivos

En esta tarea se realiza la instalación, configuración y comprobación de los sistemas operativos y servicios esenciales para nuestro entorno de pruebas individual, así como la conectividad de red, la virtualización y el sistema centralizado de copias de seguridad.

## 2. Desarrollo Obligatorio (Apto)

### 2.1. Servidores de Comunicaciones y Red

* **Sistemas implementados:** MikroTik, OPNsense y Debian con nftables (`debcomm`).

* **Configuración:** Asignación de usuarios de administración, configuración de interfaces WAN/LAN y reglas de filtrado para proporcionar acceso a Internet a la red interna.

**Evidencias:**

* Configuración de red en MikroTik:
  ![MikroTik Red](../evidencias/t5-dario_microtik.png)

* Copia de seguridad / baja MikroTik:
  ![Backup MikroTik](../evidencias/t5-backup_MicroTik.png)

* Configuración en OPNsense:
  ![OPNsense Red](../evidencias/t5-dario_opnsense.png)

* Configuración en Debian (`debcomm`):
  ![Debian Comunicacions](../evidencias/t5-dario_debcomm.png)

### 2.2. Servidor de Almacenamiento y NAS (`storesrvv`)

* **Sistema:** TrueNAS.

* **Configuración:** Usuarios de administración, parámetros de red, configuración de discos adicionales y creación de datasets/pools para almacenar las copias de seguridad del entorno.

**Evidencias:**

* Disco adicional en VirtualBox:
  ![Disco store-srvv](../evidencias/t5-disco_storesrvv_vbox.png)

* Red de store-srvv:
  ![Red store-srvv](../evidencias/t5-network_storesrvv.png)

* Creación de pool y dataset:
  ![Dataset store-srvv](../evidencias/t5-dataset_storesrvv.png)

* Verificación de las copias:
  ![Verificació còpies](../evidencias/t5-verificacion_backups_storesrvv.png)

### 2.3. Clúster de Virtualización (`Proxmox VE - PX`)

* **Nodos:** N1-PX, N2-PX, N3-PX.

* **Tareas:** Configuración de red de los nodos, creación del clúster, despliegue de máquinas virtuales y pruebas de migración conectado a TrueNAS.

**Evidencias:**

* Estado del clúster:
  ![Clúster Proxmox](../evidencias/t5-proxmox_cluster.png)

* Servidores virtuales creados:
  ![VMs Proxmox](../evidencias/t5-proxmox_servidores_creados.png)

* Prueba de migración:
  ![Migració Proxmox](../evidencias/t5-proxmox_migracion.png)

* Copias en TrueNAS:
  ![Backups Proxmox](../evidencias/t5-proxmox_backups_truenas.png)

### 2.4. Centralización de Dominio (`win-srvv` y `win-cli`)

* **Sistema:** Windows Server y Windows 11 (Client).

* **Tareas:** Creación del dominio, configuración de usuarios e integración de los clientes en la red corporativa.

**Evidencias:**

* Configuración de red en Windows Server:
  ![Red Win-Srv](../evidencias/t5-dario_win-srvv.png)

* Integración del cliente en el dominio:
  ![Client en domini](../evidencias/t5-win-cli_dominio.png)

### 2.5. Servidores de Infraestructura y Monitorización (`srv-monn` y `deb-cli`)

* **Tareas:** Configuración de sistemas Linux, herramientas de supervisión y el entorno cliente de pruebas `deb-cli`.

**Evidencias:**

* Herramientas de monitorización en `srv-monn`:
  ![Eines srv-monn](../evidencias/t5-herramientas_srv-monn.png)

* Red de `deb-cli`:
  ![Red deb-cli](../evidencias/t5-red_deb-cli.png)

## 3. Copias de Seguridad y Respaldos (Backups)

Se han recogido y verificado las copias de seguridad de todos los sistemas críticos hacia el servidor de almacenamiento centralizado:

* Respaldo de `deb-cli`: 
  ![Backup deb-cli](../evidencias/t5-backup_deb-cli.png)

* Respaldo de `debcomm`: 
  ![Backup debcomm](../evidencias/t5-backup_debcomm.png)

* Respaldo de MikroTik: 
  ![Backup MikroTik](../evidencias/t5-backup_MicroTik.png)

* Respaldo de `srv-monn`: 
  ![Backup srv-monn](../evidencias/t5-backup_srv-monn.png)

* Respaldo de `win-cli`: 
  ![Backup win-cli](../evidencias/t5-backup_win-cli.png)

* Respaldo de `win-srvv`: 
  ![Backup win-srvv](../evidencias/t5-backup_win-srvv.png)

## 4. Ampliación (Apte+) \[PENDIENTE / EN CURS\]

* **Estado:** Pendientes de concretar con el profesorado los métodos de automatización avanzada de las copias de seguridad (scripts de automatización y programación mediante cron o tareas planificadas) y su recuperación automática.
