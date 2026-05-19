# ADR-012 — Implementar Monitoreo y Logs Centralizados

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El sistema está compuesto por múltiples microservicios distribuidos en diferentes ambientes (`dev`, `qa`, `main`). A medida que aumentó la complejidad del proyecto, identificar errores y analizar el comportamiento de los servicios comenzó a ser más difícil utilizando únicamente logs locales de Docker o revisiones manuales de contenedores.

Era necesario definir una estrategia centralizada para recopilar métricas, monitorear el estado de los servicios y consolidar logs del sistema.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Logs locales por contenedor | Revisar manualmente logs con `docker logs` |
| Monitoreo básico del servidor | Supervisar únicamente CPU, RAM y disco |
| Plataforma centralizada de monitoreo | Centralizar métricas y logs de todos los servicios |

## Decisión

Se eligió implementar una solución de **monitoreo y logs centralizados** utilizando herramientas compatibles con Docker y microservicios.

La solución contempla:
- Recolección centralizada de logs
- Monitoreo de contenedores y servicios
- Visualización de métricas
- Alertas básicas ante fallos del sistema

Las herramientas consideradas incluyen:
- Prometheus
- Grafana
- Loki
- ELK Stack

## Justificación

- Facilita la detección temprana de errores y cuellos de botella.
- Permite visualizar el estado general del sistema desde un único punto.
- Mejora la trazabilidad entre servicios distribuidos.
- Simplifica el diagnóstico de fallos en ambientes DEV, QA y MAIN.
- Facilita futuras estrategias de escalabilidad y alta disponibilidad.
- Se integra correctamente con Docker, Jenkins y AWS.

## Consecuencias

- **Positivas:** Mayor observabilidad del sistema, mejor capacidad de diagnóstico y monitoreo centralizado de servicios.
- **Negativas:** Incrementa la complejidad de infraestructura y agrega nuevos servicios que deben configurarse y mantenerse.