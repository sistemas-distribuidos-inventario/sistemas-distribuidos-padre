# ADR-007 — Organizar los Repositorios dentro de una Organización de GitHub

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El proyecto está compuesto por múltiples repositorios (frontend, backends, configuración Docker, base de datos, documentación, etc.). Inicialmente los repositorios estaban alojados en cuentas personales, lo que dificultaba la administración centralizada de permisos, visibilidad y colaboración entre los integrantes del equipo.

Era necesario definir una estrategia para centralizar la gestión del proyecto y facilitar el trabajo colaborativo.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Repositorios en cuentas personales | Cada desarrollador administra sus propios repositorios |
| Monorepo único | Todo el sistema en un solo repositorio |
| Organización en GitHub | Repositorios separados administrados desde una organización central |

## Decisión

Se eligió crear una **organización en GitHub** para centralizar todos los repositorios del proyecto.

La organización contiene:
- Repositorios de frontend y backend
- Repositorios de configuración Docker y despliegue
- Repositorio de documentación
- Repositorios auxiliares para scripts y automatización

## Justificación

- Centraliza la administración de permisos y accesos.
- Facilita la colaboración entre desarrolladores sin depender de cuentas personales.
- Permite aplicar configuraciones homogéneas (branches protegidas, reglas de merge, secrets, webhooks).
- Mantiene la separación entre servicios sin perder trazabilidad global del sistema.
- Facilita futuras integraciones con Jenkins, AWS y herramientas de CI/CD.

## Consecuencias

- **Positivas:** Administración centralizada, mejor organización del proyecto, facilidad para integrar automatizaciones y pipelines.
- **Negativas:** Requiere configuración inicial de roles, permisos y políticas dentro de la organización.