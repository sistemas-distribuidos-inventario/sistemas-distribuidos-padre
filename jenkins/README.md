# HU-009 — Despliegues Automáticos con Jenkins

## Estructura de archivos entregados

```
HU-009/
├── Jenkinsfile                                        ← Pipeline declarativo principal
├── jenkins/
│   ├── Dockerfile                                     ← Imagen Jenkins con Docker CLI + Python
│   ├── docker-compose.yml                             ← Levanta Jenkins
│   └── plugins.txt                                    ← Plugins pre-instalados
├── sistemas-distribuidos-backend-1/
│   └── tests/
│       ├── conftest.py                                ← Fixtures pytest (auth)
│       └── test_auth.py                               ← Pruebas unitarias autenticación
└── sistemas-distribuidos-backend-2/
    └── tests/
        ├── conftest.py                                ← Fixtures pytest (inventario)
        └── test_inventario.py                         ← Pruebas unitarias inventario
```

---

## 1. Pre-requisitos en el servidor

- Docker Engine ≥ 24
- Docker Compose v2
- Puerto 8080 disponible (Jenkins UI)
- Puerto 50000 disponible (agentes Jenkins)

---

## 2. Levantar Jenkins

```bash
# Desde la raíz del repositorio
cd jenkins/

# Crea la red compartida si no existe
docker network create red-inventario

# Construye y levanta Jenkins
docker compose up -d --build

# Verifica que arrancó
docker logs -f jenkins-inventario
```

Abre **http://localhost:8080** en el navegador.

### Contraseña inicial (primer arranque)

```bash
docker exec jenkins-inventario cat /var/jenkins_home/secrets/master.key
```

---

## 3. Crear el Pipeline en Jenkins

1. **New Item** → nombre: `inventario-ci` → tipo: **Pipeline**
2. En la sección **Pipeline**:
   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: URL de tu repositorio
   - Branch: `*/main` (o la rama que uses)
   - Script Path: `Jenkinsfile`
3. **Save**

### Configurar webhook (opcional, recomendado)

En tu repositorio Git (GitHub/GitLab):  
`Settings → Webhooks → Add webhook`  
URL: `http://<IP_SERVIDOR>:8080/github-webhook/`  
Content type: `application/json`  
Evento: `push`

Sin webhook, el pipeline usa **polling cada minuto** (`pollSCM('* * * * *')`).

---

## 4. Estructura del pipeline

```
Preparación
    └─► Pruebas: Autenticación  ──┐
    └─► Pruebas: Inventario     ──┤  (en paralelo los builds si pasan)
                                  ▼
                        Build: Imágenes Docker
                          (4 builds en paralelo)
                                  ▼
                             Despliegue
                                  ▼
                            Health Check
                                  ▼
                              Limpieza
```

| Etapa | Qué hace |
|---|---|
| **Preparación** | Crea la red Docker si no existe, imprime info del commit |
| **Pruebas** | Ejecuta `pytest` con SQLite en memoria, publica reporte JUnit |
| **Build** | `docker build` en paralelo para los 4 módulos |
| **Despliegue** | Detiene contenedores anteriores, levanta los nuevos |
| **Health Check** | `curl` a `/health` de cada backend; falla y hace rollback si no responde |
| **Limpieza** | Elimina imágenes Docker de más de 24 h |

---

## 5. Ejecutar pruebas localmente

```bash
# Autenticación
cd sistemas-distribuidos-backend-1
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest tests/ -v

# Inventario
cd sistemas-distribuidos-backend-2
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest tests/ -v
```

---

## 6. Ver resultados del pipeline

- **Dashboard Jenkins** → `inventario-ci` → **Stage View** muestra cada etapa con duración y estado.
- **Test Results** → publicados automáticamente en la pestaña `Test Result` de cada build.
- **Console Output** → log completo con timestamps.
- Los builds fallidos hacen **rollback automático** de los contenedores y quedan marcados en rojo.

---

## 7. Notas de seguridad para producción

- Mover credenciales de BD a **Jenkins Credentials** (tipo `Secret text`) y referenciarlas con `credentials('id')` en el Jenkinsfile.
- Cambiar `POSTGRES_PASSWORD` y `SECRET_KEY` por valores reales vía variables de entorno de Jenkins o un vault.
- Habilitar autenticación en la UI de Jenkins (Matrix-based security).
