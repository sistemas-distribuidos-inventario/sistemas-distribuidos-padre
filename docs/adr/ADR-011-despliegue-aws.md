
# ADR-011 — Desplegar el Proyecto en AWS

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El proyecto inicialmente se ejecutaba únicamente en entornos locales usando Docker Compose. Era necesario contar con un entorno accesible externamente que permitiera realizar pruebas integrales, demostraciones y despliegues más cercanos a producción.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Mantener despliegue local | Ejecutar el sistema solo en máquinas de desarrollo |
| VPS tradicional | Desplegar manualmente en un servidor privado |
| AWS | Utilizar infraestructura cloud escalable y administrada |

## Decisión

Se eligió desplegar el sistema en **Amazon Web Services (AWS)** utilizando instancias EC2 para ejecutar los contenedores Docker del proyecto.

La infraestructura contempla:
- Instancias EC2 para ejecución del sistema
- Security Groups para control de acceso
- Almacenamiento persistente para PostgreSQL
- Integración futura con balanceadores y monitoreo

## Justificación

- AWS proporciona infraestructura escalable y ampliamente utilizada en entornos reales.
- Facilita la disponibilidad remota del sistema.
- Permite futuras mejoras como autoescalado, monitoreo y balanceo de carga.
- Compatible con la arquitectura basada en Docker ya implementada.
- Facilita pruebas de despliegue continuo desde Jenkins.

## Consecuencias

- **Positivas:** Sistema accesible remotamente, infraestructura escalable y mayor acercamiento a un entorno de producción real.
- **Negativas:** Incremento en costos operativos y necesidad de administrar aspectos de seguridad, redes y monitoreo cloud.