# ADR-009 — Agregar Nuevas Entidades al Modelo de Datos

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

A medida que el sistema evolucionó surgió la necesidad de soportar nuevas funcionalidades relacionadas con auditoría, trazabilidad, roles avanzados y almacenamiento de información adicional.

El modelo de datos original no contemplaba estas capacidades, por lo que era necesario extender el esquema existente mediante nuevas entidades relacionadas.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Sobrecargar tablas existentes | Agregar múltiples columnas nuevas a tablas actuales |
| Uso de campos JSON | Guardar información adicional sin nuevas tablas |
| Crear nuevas entidades relacionales | Incorporar tablas especializadas y relacionadas |

## Decisión

Se eligió **crear nuevas entidades relacionales** para soportar las nuevas funcionalidades del sistema.

Entre las entidades agregadas se incluyen:
- Auditoría y trazabilidad
- Historial de operaciones
- Gestión avanzada de roles y permisos
- Registro de eventos del sistema

## Justificación

- Mantiene el principio de separación de responsabilidades.
- Facilita consultas complejas y reportes futuros.
- Evita tablas excesivamente grandes y difíciles de mantener.
- Permite aplicar restricciones e integridad referencial adecuadamente.
- Mejora la escalabilidad del modelo de datos.

## Consecuencias

- **Positivas:** Modelo más flexible, extensible y alineado con buenas prácticas relacionales.
- **Negativas:** Incrementa la complejidad del esquema y la cantidad de joins necesarios en algunas consultas.