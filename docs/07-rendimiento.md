# Análisis de Rendimiento y Diagnóstico - Tasca 7 (ASIX ThinkLab)

| Datos de Configuración | Detalle |
| :--- | :--- |
| **Alumno** | Darío Martiáñez Pérez |
| **Número de PC** | PC-26 |
| **Red Interna (LAN)** | `192.168.26.0/24` |
| **Módulos** | ISO / FDM |

---

## 1. Descripción del Entorno y Metodología

El objetivo de esta actividad es evaluar el consumo de recursos físicos del equipo anfitrión (*host*) (**PC-26**, dotado con procesador Intel Core Ultra 5 125H de 14 núcleos / 18 hilos y $30,6\text{ GiB}$ de memoria RAM disponible) bajo distintos escenarios de carga de virtualización en **VirtualBox**.

Para ello, se han llevado a cabo tres fases de medición empírica utilizando comandos nativos de Linux (`free -h`, `top` y `htop`):
1. **Línea base en reposo:** Consumo nativo del host sin máquinas virtuales activas.
2. **Aislamiento de carga:** Impacto individual al ejecutar una única VM exigente (`store-srvv`).
3. **Concurrencia y saturación:** Encendido simultáneo progresivo de las máquinas virtuales del proyecto hasta evaluar el umbral operativo.

---

## 2. Análisis Comparativo de Recursos del Host

A continuación se presenta la tabla comparativa con las métricas reales obtenidas en cada escenario de prueba:

| Métrica del Host | Escenario 1: Reposo (Línea Base) | Escenario 2: VM Aislada (`store-srvv`) | Escenario 3: Concurrencia (VMs activas incl. debian-dades y debian-serveis) |
| :--- | :---: | :---: | :---: |
| **Máquinas Virtuales Activas** | `0` | `1` (`store-srvv` - TrueNAS) | (`N1-PX`, `N2-PX`, `N3-PX`, `MikroTik`, `win-srvv`, `store-srvv`, `srv-monn`, `deb-cli`, `win-cli`, `debian-dades`, `debian-serveis`) |
| **Memoria RAM Usada** | $3,3\text{ GiB} / 30\text{ GiB}$ ($11\%$) | $7,22\text{ GiB} / 30,6\text{ GiB}$ ($23,6\%$) | $23,7\text{ GiB} / 30,6\text{ GiB}$ ($77,5\%$) |
| **Memoria RAM Disponible** | $27\text{ GiB}$ | $23,38\text{ GiB}$ | $6,9\text{ GiB}$ |
| **Uso de Memoria Swap** | $0\text{ B} / 3,0\text{ GiB}$ | $0\text{ K} / 3,00\text{ GiB}$ | $0\text{ K} / 3,00\text{ GiB}$ |
| **Carga Media CPU (*Load Average*)** | `0.70, 0.43, 0.31` | `0.77, 0.71, 0.60` | `2.76, 1.62, 0.97` |
| **Porcentaje CPU en Reposo (`id`)** | $98,5\%$ libre | $\approx 90\%$ libre (picos puntuales en hilo 1) | $\approx 85\% - 90\%$ libre (carga repartida en 18 hilos) |

---

## 3. Evidencias de Ejecución por Escenario

### A. Línea Base en Reposo (`0` VMs)
El sistema operativo anfitrión arrancado desde el SSD externo mantiene un consumo base muy reducido ($3,3\text{ GiB}$ de RAM y más del $98\%$ de CPU inactiva), dejando prácticamente la totalidad de los recursos libres para la hipervisión.

![Consumo del Host en Reposo](../evidencias/t7_host_reposo.png)

---

### B. Impacto Individual por Máquina Virtual (`store-srvv`)
Al encender de forma aislada la máquina **TrueNAS SCALE (`store-srvv`)**, la memoria RAM consumida en el host asciende de $3,3\text{ GiB}$ a **$7,22\text{ GiB}$** ($+3,92\text{ GiB}$ de incremento neto). Se confirma que el almacenamiento basado en ZFS y servicios de red demanda un bloque elevado de memoria física desde el inicio, mientras que el impacto en procesador se mantiene muy bajo (*load average* de `0.77`).

![Impacto de VM Aislada en el Host](../evidencias/t7_host_vm-aislada.png)

---

### C. Prueba de Concurrencia y Límite Operativo (Concurrencia ampliada)
Se procedió a arrancar de forma simultánea el conjunto de máquinas virtuales del laboratorio, incluyendo los tres nodos del clúster Proxmox (`N1-PX`, `N2-PX`, `N3-PX`), el router `MikroTik`, los servidores `win-srvv`, `store-srvv`, `srv-monn`, `debian-dades`, `debian-serveis` y los clientes (`deb-cli` y `win-cli`).

![Prueba de Concurrencia de 9 VMs](../evidencias/t7_host_concurrencia.png)

---

## 4. Diagnóstico de Saturación y Conclusiones Técnicas

A partir de las métricas recopiladas durante las pruebas en el **PC-26**, se extraen las siguientes conclusiones:

1. **Recurso crítico que se satura primero (Memoria RAM):**
   - La **memoria RAM** es el principal cuello de botella del sistema. Mientras que los 18 hilos del procesador Intel Core Ultra 5 absorben con holgura la carga de las máquinas virtuales simultáneas (incluyendo los servicios adicionales como `debian-dades` y `debian-serveis`), el consumo de RAM escala rápidamente hasta alcanzar los **$23,7\text{ GiB}$ de $30,6\text{ GiB}$** ($77,5\%$).
2. **Máquinas virtuales más exigentes por recurso:**
   - **Mayor consumo de RAM:** `store-srvv` (TrueNAS SCALE, con $8048\text{ MB}$ asignados), `win-cli` (Windows 11) y el conjunto del clúster Proxmox (`N1-PX`, `N2-PX`, `N3-PX` con $4048\text{ MB}$ cada uno).
   - **Mayor consumo de CPU:** Los nodos de virtualización `N1-PX` ($9,2\%$) y `N3-PX` ($6,6\%$), seguidos por el servidor de monitorización `srv-monn` ($6,3\%$) y `win-srvv` ($6,2\%$).
3. **Límite máximo de máquinas concurrentes sin bloqueo:**
   - En el equipo del aula (**PC-26** con $32\text{ GB}$ de RAM), es posible mantener este volumen operativo de máquinas virtuales de forma simultánea sin que el sistema recurra a la partición *Swap* (`0K/3.00G`) ni sufra bloqueos.
   - *Nota comparativa:* En el equipo de casa ($16\text{ GB}$ de RAM) o en equipos limitados a $8\text{ GB}$, el límite operativo se reduce drásticamente a **2 o 3 máquinas simultáneas** dependiendo de si se incluye Windows o TrueNAS.

---

## 5. Ampliación y Optimización de Recursos (Apte+)

Para superar las limitaciones de consumo de recursos detectadas en la prueba de concurrencia estándar y alcanzar el nivel **Apte+**, se ha aplicado una estrategia de optimización basada en el reajuste inteligente tanto de la memoria RAM como de las vCPU en VirtualBox:

### Medidas Implementadas:
1. **Reajuste de asignación de vRAM y vCPU en el Clúster Proxmox:**
   - Se redujo la memoria RAM asignada a los tres nodos de virtualización (`N1-PX`, `N2-PX`, `N3-PX`) de **$4048\text{ MB}$ a $3072\text{ MB}$** por nodo.
   - Adicionalmente, se ajustó la asignación de procesadores virtuales (**vCPU**) de **2 a 1 vCPU** en cada uno de los tres nodos del clúster.
   - **Justificación técnica:** Al encontrarse los nodos en un entorno de pruebas anidado sin cargas masivas de producción concurrentes, 1 vCPU y $3\text{ GiB}$ de RAM por nodo resultan completamente suficientes para mantener la infraestructura del clúster activa, reduciendo la contención de hilos y la huella de memoria en el host físico.
2. **Optimización del margen operativo:**
   - Con esta doble reducción combinada, el consumo total de RAM del host durante la ejecución simultánea del entorno desciende de $23,7\text{ GiB}$ a **$21,4\text{ GiB}$**[cite: 10], ampliando el margen de memoria disponible de $6,9\text{ GiB}$ a **$9,2\text{ GiB}$**[cite: 10] y garantizando un funcionamiento holgado sin paginación (*swap*).

### Evidencia del Rendimiento Optimizado (Apte+)
La siguiente captura de pantalla muestra el monitor del sistema (`htop`) en el host **PC-26** junto al gestor de VirtualBox con las máquinas virtuales en ejecución tras aplicar las optimizaciones de RAM y vCPU:

![Host Optimizado Tasca 7 Apte+](../evidencias/t7-apte_host_optimizado.png)
