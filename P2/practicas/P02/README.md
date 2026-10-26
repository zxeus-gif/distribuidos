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

### Tabla 2. Disponibilidad compuesta del proyecto

| Componente | SLA individual | SLA redundante | En serie o paralelo | Costo extra de redundancia (USD/mes) |
|---|---:|---:|---|---:|
| DNS | | | | |
| Balanceador | | | | |
| Cómputo | | | | |
| Base de datos | | | | |
| Objetos | | | | |
| Compuesto sin / con redundancia | | | | |
| Minutos al mes sin / con redundancia | | | | |

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
