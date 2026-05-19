# ADR-008 — Refactorización del Esquema de Base de Datos

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El esquema inicial de la base de datos fue construido rápidamente para validar funcionalidades básicas del sistema. Con el crecimiento del proyecto aparecieron problemas de mantenibilidad, redundancia de datos y relaciones poco normalizadas.

Era necesario reorganizar la estructura del modelo relacional para soportar futuras funcionalidades y facilitar el mantenimiento del sistema.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Mantener el esquema actual | Continuar agregando tablas y columnas sobre el diseño existente |
| Reescribir completamente la base de datos | Crear un nuevo esquema desde cero |
| Refactorización progresiva | Mejorar el esquema actual mediante migraciones controladas |

## Decisión

Se eligió realizar una **refactorización progresiva de la base de datos** utilizando migraciones versionadas con Liquibase.

Los cambios incluyen:
- Normalización de tablas
- Separación de responsabilidades entre entidades
- Eliminación de redundancias
- Mejora de claves foráneas y restricciones
- Estandarización de nombres y convenciones

## Justificación

- Minimiza el impacto sobre funcionalidades ya implementadas.
- Permite evolucionar el esquema sin perder datos existentes.
- Aprovecha la infraestructura de migraciones ya definida en ADR-003.
- Facilita la mantenibilidad y escalabilidad futura del sistema.
- Reduce inconsistencias y duplicación de información.

## Consecuencias

- **Positivas:** Base de datos más mantenible, mejor integridad de datos y mayor claridad en las relaciones entre entidades.
- **Negativas:** Requiere crear y validar migraciones cuidadosamente para evitar incompatibilidades con versiones anteriores.