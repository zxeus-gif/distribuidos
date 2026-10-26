# Práctica 2 — Acuerdo de nivel de servicio: análisis de un SLA real y diseño de SLOs

## 1. Introducción

Pendiente.

## 2. Arquitectura del proyecto

Pendiente: proveedor, región, servicios y diagrama de arquitectura.

## 3. Análisis de SLA

### Tabla 1. Anatomía del SLA

| Integrante | Servicio y URL (fecha) | Compromiso por configuración | Definición de inactividad | Exclusiones | Créditos | Reclamación | Min/mes permitidos |
|---|---|---|---|---|---|---|---|
| Roberto | Azure Virtual Machines — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | 2+ VM en 2+ Zonas de Disponibilidad: 99.99%; Conjunto de Disponibilidad: 99.95%; VM de instancia única con SSD Premium/Premium v2/Ultra: 99.9%; Standard SSD: 99.5%; Standard HDD: 95% | Minutos acumulados del período aplicable en los que no existe conectividad de la máquina virtual mediante TCP o UDP. | Factores fuera del control razonable de Microsoft; hardware/software/servicios de terceros; versiones preview/beta/evaluación; configuraciones no admitidas; acciones iniciadas por el cliente. | Para 2+ zonas: <99.99% = 10%, <99% = 25%, <95% = 100%. Para VM con SSD Premium: <99.9% = 10%, <99% = 25%, <95% = 100%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente; incluir descripción, hora/duración, recursos afectados, usuarios afectados y errores. | 99.99% = 4.32 min; 99.95% = 21.6 min; 99.9% = 43.2 min; 99.5% = 216 min. |
| Roberto | Azure SQL Database — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | Implementación redundante por zona: 99.995%; Hiperescala/Crítico para la Empresa/Premium sin redundancia zonal: 99.99%; Básico o Estándar: 99.99% | Un minuto se considera no disponible si fallan todos los intentos continuos del cliente para establecer una conexión con la base de datos durante ese minuto. | Aplican las limitaciones generales del SLA: factores externos a Microsoft; servicios de terceros; preview/beta; configuraciones no admitidas; acciones iniciadas por el cliente, entre otras. | <99.995% o <99.99%, según configuración = 10%; <99% = 25%; <95% = 100%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente con los datos requeridos para validarlo. | 99.995% = 2.16 min; 99.99% = 4.32 min. |
| Roberto | Azure Storage / Blob Storage — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | Nivel Frecuente: LRS/ZRS/GRS/GZRS lectura y escritura 99.9%; RA-GRS/RA-GZRS lectura 99.99%. Para el proyecto se propone Blob Storage Frecuente con ZRS: 99.9%. | La disponibilidad se determina a partir de la tasa de transacciones de almacenamiento erróneas. Una transacción es errónea cuando no se completa dentro del tiempo máximo de procesamiento establecido para su tipo. | Se excluyen, entre otras, transacciones con errores de autenticación, operaciones que exceden cuotas, creación/eliminación de contenedores/colas/tablas y ciertas copias entre cuentas; también aplican las limitaciones generales del SLA. | Para LRS/ZRS/GRS/GZRS en nivel Frecuente: <99.9% = 10%, <99% = 25%. Para lecturas RA-GRS/RA-GZRS: <99.99% = 10%, <99% = 25%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente con la evidencia necesaria. | ZRS 99.9% = 43.2 min; RA-GRS/RA-GZRS lectura 99.99% = 4.32 min. |

## 4. Disponibilidad compuesta

### 4.1 Cadena de dependencias

Para el proyecto de tienda en línea se considera la siguiente cadena de dependencias:

```text
Usuario
  ↓
Azure Standard Load Balancer
  ↓
Azure Virtual Machines
  ↓
Azure SQL Database
  ↓
Azure Blob Storage
```

Se analizaron dos escenarios:

**Escenario 1 — Arquitectura base sin redundancia**

- Azure Standard Load Balancer.
- 1 Azure Virtual Machine con SSD Premium.
- Azure SQL Database no redundante.
- Azure Blob Storage con ZRS.

**Escenario 2 — Arquitectura mejorada con redundancia**

- Azure Standard Load Balancer.
- 2 Azure Virtual Machines distribuidas en 2 Zonas de Disponibilidad.
- Azure SQL Database con redundancia por zona.
- Azure Blob Storage con ZRS.

Los componentes principales se modelan en serie; en el escenario mejorado, la capa de cómputo utiliza dos máquinas virtuales en zonas distintas como configuración redundante.

### 4.2 Fórmula utilizada

Para componentes en serie, la disponibilidad compuesta se obtiene multiplicando las disponibilidades individuales:

```text
Acompuesta = a1 × a2 × a3 × ... × an
```

Para convertir la disponibilidad obtenida en minutos máximos de indisponibilidad en una ventana de 30 días:

```text
Minutos de inactividad = (1 - disponibilidad) × 43,200
```

### 4.3 Escenario 1 — Sin redundancia

SLA considerados:

- Azure Standard Load Balancer = 99.99%.
- Azure Virtual Machine individual con SSD Premium = 99.9%.
- Azure SQL Database no redundante = 99.99%.
- Azure Blob Storage ZRS = 99.9%.

Cálculo:

```text
A = 0.9999 × 0.999 × 0.9999 × 0.999
A = 0.99780140978001
A = 99.7801%
```

Minutos máximos de inactividad:

```text
(1 - 0.99780140978001) × 43,200 = 94.98 minutos
```

Por lo tanto, la arquitectura base tiene una disponibilidad compuesta aproximada de **99.7801%**, equivalente a **94.98 minutos de indisponibilidad al mes**.

### 4.4 Escenario 2 — Con redundancia

SLA considerados:

- Azure Standard Load Balancer = 99.99%.
- Azure Virtual Machines en 2 Zonas de Disponibilidad = 99.99%.
- Azure SQL Database con redundancia por zona = 99.995%.
- Azure Blob Storage ZRS = 99.9%.

Cálculo:

```text
A = 0.9999 × 0.9999 × 0.99995 × 0.999
A = 0.9987502699795006
A = 99.8750%
```

Minutos máximos de inactividad:

```text
(1 - 0.9987502699795006) × 43,200 = 53.99 minutos
```

Por lo tanto, la arquitectura redundante tiene una disponibilidad compuesta aproximada de **99.8750%**, equivalente a **53.99 minutos de indisponibilidad al mes**.

### Tabla 2. Disponibilidad compuesta del proyecto

| Componente | SLA individual | SLA redundante | En serie o paralelo | Costo extra de redundancia (USD/mes) |
|---|---:|---:|---|---:|
| Azure Standard Load Balancer | 99.99% | 99.99% | Serie | Pendiente |
| Cómputo — Azure Virtual Machines | 99.9% | 99.99% | Serie / Paralelo entre zonas | Pendiente |
| Base de datos — Azure SQL Database | 99.99% | 99.995% | Serie | Pendiente |
| Objetos — Azure Blob Storage | 99.9% | 99.9% | Serie | Pendiente |
| **Compuesto sin / con redundancia** | **99.7801%** | **99.8750%** | — | — |
| **Minutos al mes sin / con redundancia** | **94.98 min** | **53.99 min** | — | — |

### 4.5 Análisis de resultados

- La arquitectura sin redundancia ofrece una disponibilidad compuesta aproximada de **99.7801%**, equivalente a **94.98 minutos** de inactividad potencial al mes.
- La arquitectura con redundancia mejora a **99.8750%**, equivalente a **53.99 minutos** de inactividad potencial al mes.
- La redundancia reduce aproximadamente **40.99 minutos** de indisponibilidad potencial al mes.
- En el escenario inicial, los componentes más débiles son **Azure Virtual Machines** y **Azure Blob Storage**, ambos con un SLA de **99.9%**.
- En el escenario mejorado, el eslabón más débil continúa siendo **Azure Blob Storage**, con **99.9%**, por lo que limita el techo de disponibilidad de la cadena completa.
- El costo adicional de redundancia queda pendiente hasta recuperar o estimar los costos mensuales de la Práctica 1.

### 4.6 Diagrama lógico de la arquitectura redundante

```text
                 Usuario
                    │
                    ▼
      Azure Standard Load Balancer
                    │
             ┌──────┴──────┐
             ▼             ▼
          VM AZ1         VM AZ2
             │             │
             └──────┬──────┘
                    ▼
        Azure SQL Database (ZR)
                    │
                    ▼
        Azure Blob Storage (ZRS)
```

- **Serie:** Load Balancer, Azure SQL Database y Azure Blob Storage.
- **Redundante/paralelo:** capa de Azure Virtual Machines entre zonas de disponibilidad.

## 5. SLOs del proyecto

### Tabla 3. SLOs

| SLO | SLI (buenos ÷ válidos) | Punto de medición | Ventana | Objetivo | Presupuesto (min) | Alertas (50 % / 100 %) | Herramienta |
|---|---|---|---|---:|---:|---|---|
| Disponibilidad | | | 30 días | | | | |
| Latencia p95 | | | 30 días | | | | |
| De dominio | | | 30 días | | | | |

## 6. Cláusula de SLA

Pendiente.

## 7. Preguntas de análisis

### Pregunta 1
Pendiente.

### Pregunta 2
Pendiente.

### Pregunta 3
Pendiente.

### Pregunta 4
Pendiente.

### Pregunta 5
Pendiente.

### Pregunta 6
Pendiente.

## 8. Conclusiones

Pendiente.

## 9. Referencias

Pendiente.

## 10. Uso de IA

Pendiente: declarar el apoyo utilizado y verificar que cada integrante pueda explicar cifras, fórmulas y cláusulas.

## 11. Contribuciones

| Integrante | Contribución | Commit |
|---|---|---|
| | | |
