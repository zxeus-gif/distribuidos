# SLOs del proyecto — Tienda en línea

Proveedor de infraestructura: **Microsoft Azure**  
Ventana de evaluación: **30 días móviles (43,200 minutos)**

La práctica define el SLI como la medición, el SLO como la meta interna sobre esa medición y el presupuesto de error como la cantidad de falla permitida dentro de la ventana. Los objetivos elegidos se mantienen por debajo del techo de disponibilidad calculado para la arquitectura del proyecto.

## SLO 1 — Disponibilidad de la API

- **Servicio:** API pública de la tienda detrás de Azure Standard Load Balancer.
- **SLI (definición):** proporción de peticiones válidas respondidas correctamente. Se consideran eventos válidos las solicitudes reales de usuarios; se excluyen health checks y solicitudes 4xx causadas por entradas inválidas del cliente. Se considera evento bueno una respuesta 2xx o 3xx sin error 5xx.
- **SLI (implementación):** `(peticiones válidas con 2xx/3xx) / (total de peticiones válidas) × 100`, usando telemetría de la aplicación en Application Insights y Azure Monitor.
- **Punto de medición:** API de la tienda / Application Insights; contraste mediante monitor sintético externo.
- **Ventana:** 30 días móviles.
- **Objetivo (SLO):** **99.5%**.
- **Presupuesto de error:** `(1 - 0.995) × 43,200 = 216 min` por ventana.
- **Alertas:** advertencia al consumir **108 min (50%)**; crítica al consumir **216 min (100%)**.
- **Política:** al llegar al 50% se revisan logs, cambios recientes y capacidad. Al llegar al 100% se congelan despliegues de funcionalidades nuevas y solo se permiten trabajos de confiabilidad, corrección, rendimiento y seguridad. Las funciones nuevas se reanudan cuando se haya recuperado al menos 25% del presupuesto en la ventana móvil.
- **Verificación externa:** monitor sintético cada 5 minutos sobre `GET /health` o un endpoint público equivalente.
- **Responsable:** Roberto como responsable operativo inicial; revisión del equipo del proyecto.
- **Revisión:** cada cierre parcial y después de incidentes relevantes.

## SLO 2 — Latencia p95 de la API

- **Servicio:** API pública de la tienda.
- **SLI (definición):** proporción de minutos válidos con tráfico en los que la latencia p95 de las peticiones válidas es menor de **500 ms**. Se excluyen health checks y errores 4xx ocasionados por solicitudes inválidas del cliente.
- **SLI (implementación):** `minutos con p95 < 500 ms / minutos válidos con tráfico × 100`, calculando el percentil 95 de duración desde Application Insights.
- **Punto de medición:** Application Insights / Azure Monitor en la API.
- **Ventana:** 30 días móviles.
- **Objetivo (SLO):** **95%** de los minutos válidos con p95 menor de 500 ms.
- **Presupuesto de error:** `(1 - 0.95) × 43,200 = 2,160 min` por ventana.
- **Alertas:** advertencia al consumir **1,080 min (50%)**; crítica al consumir **2,160 min (100%)**.
- **Política:** al alcanzar 50% se revisan consultas lentas, uso de CPU/memoria, acceso a base de datos y cambios recientes. Al alcanzar 100% se congelan nuevas funcionalidades que puedan aumentar carga y se priorizan optimizaciones de rendimiento hasta recuperar al menos 25% del presupuesto.
- **Verificación externa:** mediciones sintéticas periódicas sobre operaciones representativas de catálogo y consulta de productos.
- **Responsable:** Roberto como responsable operativo inicial; revisión del equipo del proyecto.
- **Revisión:** cada cierre parcial y después de cambios de arquitectura.

## SLO 3 — Creación correcta de órdenes

- **Servicio:** operación de creación de órdenes de la tienda (`POST /orders`).
- **SLI (definición):** proporción de intentos válidos de creación de orden que terminan correctamente. Un intento válido es una solicitud que supera la validación de entrada; se excluyen solicitudes rechazadas con 4xx por datos incorrectos del cliente. Un evento bueno es una orden persistida correctamente y confirmada con respuesta 2xx/201.
- **SLI (implementación):** `(órdenes creadas correctamente) / (intentos válidos de crear orden) × 100`, usando eventos y solicitudes de Application Insights junto con los registros de órdenes en Azure SQL Database.
- **Punto de medición:** endpoint `POST /orders`, telemetría de la API y registros de Azure SQL Database.
- **Ventana:** 30 días móviles.
- **Objetivo (SLO):** **99.0%**.
- **Presupuesto de error:** el presupuesto principal es **1% de los intentos válidos de creación de orden**. Para la tabla de la práctica se registra además el equivalente temporal: `(1 - 0.99) × 43,200 = 432 min`.
- **Alertas:** advertencia al consumir **0.5% de intentos fallidos** o su equivalente de **216 min**; crítica al consumir **1% de intentos fallidos** o **432 min** equivalentes.
- **Política:** al 50% se revisan errores de aplicación, transacciones y base de datos. Al 100% se congelan funciones nuevas relacionadas con checkout/órdenes y se prioriza la recuperación del flujo de compra. Se reanudan funciones nuevas después de recuperar al menos 25% del presupuesto.
- **Verificación externa:** prueba sintética de creación de una orden de prueba en un entorno controlado cuando exista un endpoint preparado para ello.
- **Responsable:** Roberto como responsable operativo inicial; revisión del equipo del proyecto.
- **Revisión:** cada cierre parcial y después de cambios en el flujo de órdenes.

## Política general de presupuesto de error

1. **Menos de 50% consumido:** desarrollo normal y despliegues habituales.
2. **50% consumido:** advertencia; revisión de métricas, logs, capacidad y cambios recientes. Los cambios de alto riesgo requieren revisión adicional.
3. **100% consumido:** se congelan nuevas funcionalidades. Solo se permiten correcciones de disponibilidad, rendimiento, seguridad y defectos.
4. **Recuperación:** se reanudan nuevas funciones cuando la ventana móvil haya recuperado al menos **25% del presupuesto de error** correspondiente.
5. **Responsabilidad:** el responsable operativo registra incidentes y decisiones en la bitácora del proyecto.
