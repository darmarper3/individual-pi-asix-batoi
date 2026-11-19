# PI ASIX - Individual

**Alumno:** Dario Martínez Pérez  
**Curso:** 1º ASIX  
**Centro:** CIP FP Batoi  
**Módulos:** Implantación de Sistemas Operativos (ISO) / Fundamentos de Hardware (FH)  
**PC en el Aula:** PC-26  
**Red utilizada:** `192.168.26.0/24`  

---

## Descripción del Proyecto

Este repositorio contiene la documentación técnica y las evidencias de configuración del entorno de pruebas individual desarrollado en **VirtualBox** sobre el SO de mi **SSD externo**, simulando la infraestructura de red corporativa del proyecto.

---

## Índice de Documentación

A continuación se enlazan los informes de cada una de las tareas situados en la carpeta `docs/`:

* **[Tasca 0 - Entorno Individual (Apte+)](docs/00-entorno-virtual.md):** Documentación del montaje en VirtualBox, recursos asignados a las VMs, esquema de red e incidencias técnicas resueltas.
* **[Tasca 1 - Predicciones Iniciales](docs/00-predicciones.md):** *(Opcional / En desarrollo)*
- **[Tasca 2 - Inventario y Diagnóstico (Apte+)](docs/02-inventari.md):** Inventario de hardware del PC-26 y equipo personal, comparativa de recursos, cuellos de botella y pruebas cuantitativas de E/S y red.
* **[Tasca 4 - Diseño de Red e Interconexión](docs/04-disseny.md):** *Esquema de red e interconexión (192.168.26.0/24), parámetros de interfaz, tabla de dimensionamiento de VMs, pasarelas de comunicaciones alternativas y verificación empírica de conectividad y enrutamiento.*
* **[Tasca 5 - Sistemas y Copias de Seguridad](docs/05-instalacio.md):** *Implantación, configuración y verificación de pasarelas de comunicaciones, almacenamiento NAS con TrueNAS, clúster Proxmox, centralización de dominios y sistema de copias de seguridad.*
- **[Tasca 6 - Almacenamiento y RAID](docs/06-almacenamiento-raid.md):** *Implementación de RAID por software (`mdadm`) en Debian (`deb-cli`) con niveles RAID 0, RAID 1 y RAID 5, verificación de dispositivos, sistemas de ficheros (`ext4`) y persistencia.*
- **[Tasca 7 - Rendimiento y Optimización](docs/07-rendimiento-optimizacion.md):** Líneas base de rendimiento, análisis de registros del sistema (`journalctl`, `vmstat`, `iostat`), pruebas de carga, estudio de saturación de recursos y optimización en máquinas virtuales (*Apte+*).

---

## Organización del Repositorio

* `docs/`: Documentos en Markdown con los informes detallados de cada tarea.
* `evidencias/`: Capturas de pantalla, archivos de configuración, logs y el diagrama de red.
