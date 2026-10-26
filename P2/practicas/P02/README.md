# Práctica 2 — Acuerdo de nivel de servicio: análisis de un SLA real y diseño de SLOs

## 1. Introducción

Pendiente.

## 2. Arquitectura del proyecto

Pendiente: proveedor, región, servicios y diagrama de arquitectura.

## 3. Análisis de SLA

### Tabla 1. Anatomía del SLA

| Integrante | Servicio y URL (fecha) | Compromiso por configuración | Definición de inactividad | Exclusiones | Créditos | Reclamación | Min/mes permitidos |
|---|---|---|---|---|---|---|---|
| Integrante 1 | Azure Virtual Machines — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | 2+ VM en 2+ Zonas de Disponibilidad: 99.99%; Conjunto de Disponibilidad: 99.95%; VM de instancia única con SSD Premium/Premium v2/Ultra: 99.9%; Standard SSD: 99.5%; Standard HDD: 95% | Minutos acumulados del período aplicable en los que no existe conectividad de la máquina virtual mediante TCP o UDP. | Factores fuera del control razonable de Microsoft; hardware/software/servicios de terceros; versiones preview/beta/evaluación; configuraciones no admitidas; acciones iniciadas por el cliente. | Para 2+ zonas: <99.99% = 10%, <99% = 25%, <95% = 100%. Para VM con SSD Premium: <99.9% = 10%, <99% = 25%, <95% = 100%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente; incluir descripción, hora/duración, recursos afectados, usuarios afectados y errores. | 99.99% = 4.32 min; 99.95% = 21.6 min; 99.9% = 43.2 min; 99.5% = 216 min. |
|  Integrante 2 | Azure SQL Database — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | Implementación redundante por zona: 99.995%; Hiperescala/Crítico para la Empresa/Premium sin redundancia zonal: 99.99%; Básico o Estándar: 99.99% | Un minuto se considera no disponible si fallan todos los intentos continuos del cliente para establecer una conexión con la base de datos durante ese minuto. | Aplican las limitaciones generales del SLA: factores externos a Microsoft; servicios de terceros; preview/beta; configuraciones no admitidas; acciones iniciadas por el cliente, entre otras. | <99.995% o <99.99%, según configuración = 10%; <99% = 25%; <95% = 100%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente con los datos requeridos para validarlo. | 99.995% = 2.16 min; 99.99% = 4.32 min. |
|  Integrante 3 | Azure Storage / Blob Storage — [SLA oficial de Microsoft](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services?lang=31), versión 01/10/2026 | Nivel Frecuente: LRS/ZRS/GRS/GZRS lectura y escritura 99.9%; RA-GRS/RA-GZRS lectura 99.99%. Para el proyecto se propone Blob Storage Frecuente con ZRS: 99.9%. | La disponibilidad se determina a partir de la tasa de transacciones de almacenamiento erróneas. Una transacción es errónea cuando no se completa dentro del tiempo máximo de procesamiento establecido para su tipo. | Se excluyen, entre otras, transacciones con errores de autenticación, operaciones que exceden cuotas, creación/eliminación de contenedores/colas/tablas y ciertas copias entre cuentas; también aplican las limitaciones generales del SLA. | Para LRS/ZRS/GRS/GZRS en nivel Frecuente: <99.9% = 10%, <99% = 25%. Para lecturas RA-GRS/RA-GZRS: <99.99% = 10%, <99% = 25%. Con 99.0% sobre USD 100 corresponde USD 10. | Reclamación a soporte de Microsoft dentro de 60 días posteriores al incidente con la evidencia necesaria. | ZRS 99.9% = 43.2 min; RA-GRS/RA-GZRS lectura 99.99% = 4.32 min. |

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

## 5. SLIs, SLOs y presupuesto de error

La práctica solicita definir entre dos y tres SLIs con eventos válidos y buenos claramente especificados, punto de medición, ventana móvil de 30 días, objetivo realista, presupuesto de error y alertas al 50% y al 100%. Para la tienda se seleccionan disponibilidad de la API, latencia p95 y creación de órdenes.

### Tabla 3. SLOs

| SLO | SLI (buenos ÷ válidos) | Punto de medición | Ventana | Objetivo | Presupuesto (min) | Alertas (50 % / 100 %) | Herramienta |
|---|---|---|---|---:|---:|---|---|
| Disponibilidad de la API | Peticiones válidas con respuesta 2xx/3xx ÷ peticiones válidas. Se excluyen health checks y errores 4xx provocados por el cliente. | API / telemetría de aplicación y verificación externa | 30 días móviles | **99.5%** | **216 min** | **108 / 216 min** | Azure Monitor + Application Insights + monitor sintético externo |
| Latencia p95 | Minutos válidos con tráfico en los que la latencia p95 de las peticiones válidas es < 500 ms ÷ minutos válidos con tráfico | API / Application Insights | 30 días móviles | **95%** | **2,160 min** | **1,080 / 2,160 min** | Azure Monitor + Application Insights |
| Órdenes creadas con éxito | Intentos válidos de creación de orden que terminan correctamente ÷ intentos válidos de creación de orden. Se excluyen solicitudes inválidas rechazadas con 4xx. | Endpoint `POST /orders` y base de datos de órdenes | 30 días móviles | **99.0%** | **432 min equivalentes** | **216 / 432 min** | Application Insights + registros de la API / Azure SQL |

### 5.1 SLO de disponibilidad

Se adopta un objetivo interno de **99.5%**, menor que el techo contractual compuesto de la arquitectura base (**99.7801%**) y suficientemente separado de él para dejar margen a fallas propias de la aplicación, despliegues y errores de configuración.

```text
Presupuesto = (1 - 0.995) × 43,200
            = 216 minutos
```

- Al consumir **108 min (50%)**, se genera advertencia y se revisan errores, cambios recientes y capacidad.
- Al consumir **216 min (100%)**, se congelan despliegues de funciones nuevas y solo se permiten correcciones de confiabilidad.

### 5.2 SLO de latencia p95

Para evitar que una aplicación técnicamente disponible sea demasiado lenta para el usuario, se define como intervalo bueno aquel minuto con tráfico en el que el percentil 95 de latencia de las peticiones válidas es menor a **500 ms**.

El objetivo es que al menos **95%** de los minutos válidos de la ventana cumplan el umbral.

```text
Presupuesto = (1 - 0.95) × 43,200
            = 2,160 minutos
```

- Advertencia al consumir **1,080 min**.
- Alerta crítica al consumir **2,160 min**.

### 5.3 SLO de dominio — creación de órdenes

El SLI de dominio mide la función principal de la tienda: que una orden válida sea creada correctamente. Se consideran válidos los intentos que pasan la validación de entrada; los errores 4xx ocasionados por datos incorrectos del cliente no se cuentan como fallas del sistema.

El objetivo es **99.0% de órdenes creadas correctamente**. Para la tabla se registra el equivalente temporal del presupuesto solicitado por la práctica:

```text
Presupuesto equivalente = (1 - 0.99) × 43,200
                         = 432 minutos
```

Operativamente, al ser un SLI basado en eventos, el presupuesto real también significa que como máximo **1% de los intentos válidos de creación de orden** pueden fallar dentro de la ventana.

- Advertencia al equivalente del **50%: 216 min** o al consumir 0.5% de intentos válidos fallidos.
- Alerta crítica al equivalente del **100%: 432 min** o al alcanzar 1% de intentos válidos fallidos.

### 5.4 Política del presupuesto de error

La política común para los tres SLOs será la siguiente:

1. Mientras el consumo sea menor al 50%, el desarrollo continúa normalmente.
2. Al alcanzar 50%, se revisan logs, métricas, cambios recientes y capacidad; los despliegues de mayor riesgo requieren revisión antes de liberarse.
3. Al alcanzar 100%, se congelan nuevas funcionalidades y únicamente se realizan correcciones de errores, rendimiento, disponibilidad o seguridad.
4. Los despliegues de funciones nuevas se reanudan cuando la ventana móvil recupere al menos **25% del presupuesto de error**.
5. El responsable operativo será **Roberto**, con revisión del equipo del proyecto cuando se incorporen los demás integrantes.

Los detalles completos de cada SLO se mantienen en [`docs/slo.md`](../../docs/slo.md).

## 6. Cláusula de SLA

### Acuerdo de Nivel de Servicio de la tienda en línea

**Compromiso mensual de disponibilidad.** El servicio de tienda en línea se compromete a mantener una disponibilidad mensual de al menos **99.0%** para su API pública y las funciones esenciales de consulta de catálogo, carrito y creación de órdenes. La disponibilidad se calculará sobre los minutos del periodo mensual aplicable a partir de las mediciones de la API y del monitor sintético externo.

**Definición de indisponibilidad.** Se considerará un minuto como no disponible cuando, durante ese minuto, todos los intentos válidos de acceso a la API pública fallen por causas atribuibles al sistema, produzcan errores HTTP 5xx o no obtengan una respuesta correcta dentro del tiempo máximo configurado por el monitor. No se consideran fallas del servicio los errores HTTP 4xx producidos por solicitudes inválidas del cliente ni los health checks internos.

**Mantenimiento y exclusiones.** No se contabilizará como indisponibilidad: (1) una ventana de mantenimiento programado de hasta **60 minutos por semana**, siempre que sea anunciada previamente; (2) fallas ocasionadas por dispositivos, redes, credenciales, configuraciones o acciones del cliente; (3) fallas de servicios externos o de terceros fuera del control razonable del equipo; (4) eventos de fuerza mayor; (5) uso abusivo, intentos que excedan cuotas o límites publicados; y (6) funciones experimentales o en versión preliminar que hayan sido identificadas como tales.

**Créditos de servicio.** Si la disponibilidad mensual medida incumple el compromiso de 99.0%, el cliente podrá solicitar un crédito sobre el importe mensual correspondiente al servicio afectado, de acuerdo con los siguientes escalones:

| Disponibilidad mensual obtenida | Crédito de servicio |
|---:|---:|
| De 98.0% a < 99.0% | 10% |
| De 95.0% a < 98.0% | 25% |
| < 95.0% | 50% |

El crédito no podrá exceder el importe pagado por el servicio afectado durante el periodo de la reclamación y constituye la compensación prevista por este SLA.

**Procedimiento de reclamación.** El cliente deberá presentar la reclamación dentro de los **30 días naturales posteriores al incidente**, indicando fecha y hora, duración estimada, función afectada, evidencia disponible y descripción del impacto. El equipo contrastará la reclamación con los registros de Application Insights, Azure Monitor y el monitor sintético. Una reclamación validada generará el crédito correspondiente al escalón de disponibilidad mensual alcanzado.

**Relación entre SLO y SLA.** El SLO interno de disponibilidad se fijó en **99.5%**, mientras que el SLA externo se establece en **99.0%**. El objetivo interno es deliberadamente más exigente para conservar un margen de seguridad que permita detectar y corregir degradaciones antes de incumplir la promesa realizada al cliente.

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
| Roberto | Análisis de SLA de Azure, disponibilidad compuesta, definición de SLIs/SLOs y redacción de la cláusula SLA del proyecto | Ver historial de commits |
