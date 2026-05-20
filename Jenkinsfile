pipeline {

    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'inventario'
        NETWORK_NAME         = 'red-inventario'
        DOCKER_BUILDKIT      = '1'

        IMG_AUTH    = "backend-autenticacion:${BUILD_NUMBER}"
        IMG_INV     = "backend-inventario:${BUILD_NUMBER}"
        IMG_FRONT   = "frontend-inventario:${BUILD_NUMBER}"
        IMG_BD      = "bd-liquibase:${BUILD_NUMBER}"
    }

    triggers {
        pollSCM('* * * * *')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        stage('Preparación') {
            steps {
                echo "==> Build #${BUILD_NUMBER} | Rama: ${GIT_BRANCH}"
                echo "==> Commit: ${GIT_COMMIT}"

                sh '''
                    docker network inspect ${NETWORK_NAME} > /dev/null 2>&1 \
                        || docker network create ${NETWORK_NAME}
                '''
            }
        }

        stage('Pruebas: Autenticación') {
            steps {
                dir('sistemas-distribuidos-backend-1') {
                    sh '''
                        python3 -m venv .venv
                        . .venv/bin/activate
                        pip install --quiet -r requirements.txt
                        pytest tests/ \
                            --tb=short \
                            --junitxml=reports/test-auth.xml \
                            -v
                    '''
                }
            }
            post {
                always {
                    junit 'sistemas-distribuidos-backend-1/reports/test-auth.xml'
                }
            }
        }

        stage('Pruebas: Inventario') {
            steps {
                dir('sistemas-distribuidos-backend-2') {
                    sh '''
                        python3 -m venv .venv
                        . .venv/bin/activate
                        pip install --quiet -r requirements.txt
                        pytest tests/ \
                            --tb=short \
                            --junitxml=reports/test-inventario.xml \
                            -v
                    '''
                }
            }
            post {
                always {
                    junit 'sistemas-distribuidos-backend-2/reports/test-inventario.xml'
                }
            }
        }

        stage('Build: Imágenes Docker') {
            steps {
                parallel(
                    'Build BD': {
                        dir('sistemas-distribuidos-bd') {
                            sh "docker build -t ${IMG_BD} ."
                        }
                    },
                    'Build Autenticación': {
                        dir('sistemas-distribuidos-backend-1') {
                            sh "docker build -t ${IMG_AUTH} ."
                        }
                    },
                    'Build Inventario': {
                        dir('sistemas-distribuidos-backend-2') {
                            sh "docker build -t ${IMG_INV} ."
                        }
                    },
                    'Build Frontend': {
                        dir('sistemas-distribuidos-frontend') {
                            sh "docker build -t ${IMG_FRONT} ."
                        }
                    }
                )
            }
        }

        stage('Despliegue') {
            steps {
                echo '==> Deteniendo contenedores anteriores...'

                sh '''
                    docker stop postgres        2>/dev/null || true
                    docker stop bd-liquibase    2>/dev/null || true
                    docker stop backend-autenticacion 2>/dev/null || true
                    docker stop backend-inventario    2>/dev/null || true
                    docker stop frontend-inventario   2>/dev/null || true

                    docker rm   postgres        2>/dev/null || true
                    docker rm   bd-liquibase    2>/dev/null || true
                    docker rm   backend-autenticacion 2>/dev/null || true
                    docker rm   backend-inventario    2>/dev/null || true
                    docker rm   frontend-inventario   2>/dev/null || true
                '''

                echo '==> Levantando base de datos...'
                dir('sistemas-distribuidos-bd') {
                    sh '''
                        docker-compose up -d postgres
                        # Espera a que Postgres esté listo (healthcheck)
                        for i in $(seq 1 30); do
                            docker inspect --format="{{.State.Health.Status}}" postgres \
                                | grep -q "healthy" && break
                            echo "Esperando postgres... intento $i/30"
                            sleep 3
                        done
                        docker-compose up bd-liquibase
                    '''
                }

                echo '==> Levantando backends y frontend...'
                sh '''
                    # Autenticación
                    docker run -d \
                        --name backend-autenticacion \
                        --network ${NETWORK_NAME} \
                        -p 5000:5000 \
                        -e SECRET_KEY=clave-secreta-dev \
                        -e DB_HOST=postgres \
                        -e DB_PORT=5432 \
                        -e DB_NAME=inventario_db \
                        -e DB_USER=admin \
                        -e DB_PASSWORD=admin123 \
                        ${IMG_AUTH}

                    # Inventario
                    docker run -d \
                        --name backend-inventario \
                        --network ${NETWORK_NAME} \
                        -p 5001:5001 \
                        -e SECRET_KEY=clave-secreta-dev \
                        -e DB_HOST=postgres \
                        -e DB_PORT=5432 \
                        -e DB_NAME=inventario_db \
                        -e DB_USER=admin \
                        -e DB_PASSWORD=admin123 \
                        ${IMG_INV}

                    # Frontend
                    docker run -d \
                        --name frontend-inventario \
                        --network ${NETWORK_NAME} \
                        -p 3000:80 \
                        ${IMG_FRONT}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 5   # Margen para que los contenedores inicien

                    echo "--- Verificando backend-autenticacion ---"
                    curl -sf http://localhost:5000/auth/health \
                        || (echo "FALLO: backend-autenticacion" && exit 1)

                    echo "--- Verificando backend-inventario ---"
                    curl -sf http://localhost:5001/inv/health \
                        || (echo "FALLO: backend-inventario" && exit 1)

                    echo "--- Verificando frontend ---"
                    curl -sf http://localhost:3000 \
                        || (echo "FALLO: frontend" && exit 1)

                    echo "Todos los servicios responden correctamente."
                '''
            }
        }

        stage('Limpieza') {
            steps {
                sh '''
                    # Elimina imágenes con tag numérico menor al actual
                    docker image prune -f --filter "until=24h" || true
                '''
            }
        }
    }

    post {

        success {
            echo """
╔══════════════════════════════════════╗
║  ✅  PIPELINE EXITOSO  Build #${BUILD_NUMBER}  ║
╚══════════════════════════════════════╝
Rama   : ${GIT_BRANCH}
Commit : ${GIT_COMMIT}
Duración: ${currentBuild.durationString}
"""
        }

        failure {
            echo """
╔══════════════════════════════════════╗
║  ❌  PIPELINE FALLIDO  Build #${BUILD_NUMBER}  ║
╚══════════════════════════════════════╝
Etapa fallida: ${currentBuild.result}
Revisa los logs en: ${BUILD_URL}
"""
            sh '''
                docker stop backend-autenticacion backend-inventario frontend-inventario 2>/dev/null || true
                docker rm   backend-autenticacion backend-inventario frontend-inventario 2>/dev/null || true
            '''
        }

        always {
            junit allowEmptyResults: true,
                  testResults: '**/reports/test-*.xml'

            cleanWs()
        }
    }
}
