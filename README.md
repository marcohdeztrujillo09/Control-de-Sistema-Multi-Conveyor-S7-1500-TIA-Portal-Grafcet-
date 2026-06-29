# 🏭 Control de Sistema Multi-Conveyor — S7-1500 (TIA Portal + Grafcet)

> TIA Portal · Grafcet · S7-1500 · PLCSIM · Factory I/O

Práctica de automatización industrial que implementa el control de un sistema multi-conveyor con lógica de prioridad, contador de producción, señalización de estado y gestión de paros mediante **Grafcet** sobre un PLC **Siemens S7-1500**, simulado en **PLCSIM** y visualizado en **Factory I/O**.

> ⚠️ **Nota:** Esta práctica fue desarrollada en un entorno educativo. El **Reset** y el selector **Auto/Manual** están presentes en el panel de Factory I/O pero **no están implementados** en esta versión del programa.

---

## 📋 Tabla de contenidos

- [Estructura del repositorio](#-estructura-del-repositorio)
- [Descripción general](#-descripción-general)
- [Funcionalidades implementadas](#-funcionalidades-implementadas)
- [Mapa de E/S — Factory I/O](#-mapa-de-es--factory-io)
- [Herramientas y entorno](#-herramientas-y-entorno)
- [Descripción del programa](#-descripción-del-programa)
- [Comportamiento del sistema](#-comportamiento-del-sistema)
- [Requisitos y compatibilidad](#-requisitos-y-compatibilidad)

---

## 📁 Estructura del repositorio

```text
├── 📁 capturas/                        # Capturas de Factory I/O y TIA Portal
├── 📁 tia-portal/                      # Proyecto exportado de TIA Portal
│   └── 📄 control_cintas.ap19          # Archivo del proyecto (TIA Portal)
├── 📁 factoryio/                       # Escena de simulación
│   └── 📄 escena_cintas.factoryio      # Escena preconfigurada de Factory I/O
```

---

## 📝 Descripción general

El sistema controla **dos cintas transportadoras independientes** (Conveyor 1 y Conveyor 2) que convergen en una zona de transferencia. Las cintas transportan palés con cajas desde sus respectivas entradas hasta una salida común. El programa gestiona la lógica de prioridad, el conteo de producción y los distintos modos de parada mediante **Grafcets** implementados en TIA Portal.

La simulación física se realiza íntegramente en **Factory I/O**, comunicada con el PLC virtual **PLCSIM** a través del driver de Factory I/O para Siemens.

---

## ✅ Funcionalidades implementadas

| Función | Descripción |
|---|---|
| **Marcha del sistema** | Pulsador Start arranca ambas cintas y el conveyor de salida |
| **Prioridad Cinta 1** | Si ambas cintas detectan carga simultáneamente, la Cinta 1 tiene preferencia sobre la Cinta 2 |
| **Contador de producción** | Cuenta el total acumulado de cajas que pasan por el sensor de salida, mostrado en el contador del panel |
| **Paro temporal** | Detiene el sistema al momento; el LED rojo del panel parpadea en intermitencia hasta que se reanuda la marcha, retomando desde el punto donde se detuvo |
| **Seta de emergencia** | Para el sistema de forma total e inmediata; el LED rojo parpadea en intermitencia y **no permite reiniciar** hasta que se desenclave manualmente |
| **LED de marcha** | Indicador verde activo cuando el sistema está en funcionamiento |
| **LED de paro** | Indicador rojo con intermitencia durante paro temporal o emergencia |

---

## 🔌 Mapa de E/S — Factory I/O

### Entradas (Sensores y Mandos)

| Dirección | Señal | Descripción |
|---|---|---|
| %I0.0 | At entry 1 | Sensor de presencia — entrada Cinta 1 |
| %I0.1 | At entry 2 | Sensor de presencia — entrada Cinta 2 |
| %I0.2 | At transfer 1 | Sensor de presencia — zona de transferencia Cinta 1 |
| %I0.3 | At transfer 2 | Sensor de presencia — zona de transferencia Cinta 2 |
| %I0.4 | At exit | Sensor de presencia — salida (dispara el contador) |
| %I0.5 | Start | Pulsador de marcha |
| %I0.6 | Reset | Pulsador de reset *(no implementado)* |
| %I0.7 | Stop | Pulsador de paro temporal |
| %I1.0 | Emergency stop | Seta de emergencia |
| %I1.1 | Auto | Selector Auto/Manual *(no implementado)* |
| %I1.2 | FACTORY I/O Running | Señal de sincronización con la simulación |

### Salidas (Actuadores y Señalización)

| Dirección | Señal | Descripción |
|---|---|---|
| %Q0.0 | Conveyor 1 | Motor Cinta 1 |
| %Q0.1 | Load 1 | Cargador de palés Cinta 1 |
| %Q0.2 | Unload 1 | Descargador Cinta 1 |
| %Q0.3 | Transfer right 1 | Transferencia derecha Cinta 1 |
| %Q0.4 | Transfer left 1 | Transferencia izquierda Cinta 1 |
| %Q0.5 | Conveyor 2 | Motor Cinta 2 |
| %Q0.6 | Load 2 | Cargador de palés Cinta 2 |
| %Q0.7 | Unload 2 | Descargador Cinta 2 |
| %Q1.0 | Transfer right 2 | Transferencia derecha Cinta 2 |
| %Q1.1 | Transfer left 2 | Transferencia izquierda Cinta 2 |
| %Q1.2 | Exit conveyor | Cinta de salida común |
| %Q1.3 | Start light | LED verde de marcha |
| %Q1.4 | Reset light | LED de reset *(no utilizado)* |
| %Q1.5 | Stop light | LED rojo de paro / emergencia (intermitencia) |
| %QD30 | Counter | Contador acumulado de cajas (DINT) |

---

## 🛠️ Herramientas y entorno

| Elemento | Tecnología |
|---|---|
| **PLC** | Siemens S7-1500 |
| **Entorno de programación** | TIA Portal V16 |
| **Lenguaje de programación** | Ladder (KOP) / Grafcet |
| **Metodología de diseño** | Grafcet |
| **Simulador PLC** | PLCSIM |
| **Simulador de planta** | Factory I/O |

---

## 💻 Descripción del programa

El programa está estructurado siguiendo la metodología **Grafcet** implementada directamente en TIA Portal. El Grafcet define las etapas y transiciones del sistema de forma visual y estructurada.

Las principales secciones del programa son:

- **Gestión de etapas Grafcet:** variables de etapa que se activan y desactivan según las condiciones de los sensores y mandos.
- **Lógica de prioridad:** comparación simultánea de `At entry 1` y `At entry 2`; si ambas están activas, la Cinta 1 bloquea el avance de la Cinta 2 hasta que libera la zona de transferencia.
- **Contador de producción:** incremento de `%QD30` en cada flanco ascendente del sensor `At exit`.
- **Gestión de paros:** dos niveles de parada con comportamiento diferenciado (paro recuperable y emergencia no recuperable) y generación de señal de intermitencia para el LED rojo.

---

## 🔄 Comportamiento del sistema

```
[START] ──► Arranque de Cinta 1 + Cinta 2 + Salida
                │
         ┌──────┴──────┐
    Carga en C1    Carga en C1 + C2
         │              │
    C1 avanza      C1 tiene prioridad
                   C2 espera hasta que C1 libera
                        │
                   C2 avanza
                        │
              [At exit] → Contador +1
                        │
              ┌──────────┴──────────┐
           [STOP]             [EMERGENCIA]
              │                    │
        Paro inmediato       Paro total
        LED rojo parpadea    LED rojo parpadea
        Retoma con START     No permite reinicio
                             hasta desenclave manual
```

---

## 📋 Requisitos y compatibilidad

- **TIA Portal V16** o superior (puede requerir migración del proyecto en versiones más nuevas)
- **PLCSIM** compatible con S7-1500
- **Factory I/O** con driver de Siemens configurado como S7-1500

> ⚠️ Si se abre el proyecto con una versión más reciente de TIA Portal, se recomienda hacer una copia de seguridad antes de migrar.

---

> Práctica de automatización industrial · Técnico en Automatización y Robótica Industrial · TIA Portal + Grafcet + Factory I/O
