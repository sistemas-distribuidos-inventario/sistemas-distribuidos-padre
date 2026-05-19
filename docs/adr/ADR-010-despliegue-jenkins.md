# ADR-010 — Despliegue Automatizado con Jenkins

**Fecha:** 2026-05-18  
**Estado:** Aceptado  
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El despliegue manual de los servicios generaba riesgo de errores humanos y hacía más lento el proceso de integración entre ambientes (`dev`, `qa`, `main`).

Era necesario implementar una herramienta de integración y despliegue continuo que automatizara la construcción, pruebas y despliegue del sistema.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Despliegue manual | Ejecutar comandos Docker y Git manualmente |
| GitHub Actions | Automatización integrada en GitHub |
| Jenkins | Servidor de automatización independiente y configurable |

## Decisión

Se eligió **Jenkins** como plataforma de integración y despliegue continuo (CI/CD).

El pipeline automatiza:
- Clonado de repositorios
- Ejecución de pruebas
- Construcción de imágenes Docker
- Despliegue por ambiente
- Validaciones básicas posteriores al despliegue

```groovy
pipeline {
    stages {
        stage('Tests') {
            steps {
                sh 'pytest'
            }
        }

        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }
}

## Justificación

- Permite automatizar completamente el flujo de despliegue.
- Jenkins es altamente configurable y compatible con Docker.
- Facilita la integración con múltiples repositorios y ambientes.
- Reduce errores manuales y mejora la reproducibilidad.
- Puede ejecutarse en infraestructura propia sin depender de servicios externos.

## Consecuencias

- **Positivas:** Despliegues automatizados, reducción de errores humanos y mayor velocidad de integración.
- **Negativas:** Requiere mantenimiento del servidor Jenkins y configuración inicial de pipelines, credenciales y agentes.
```