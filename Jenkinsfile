pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/JeanProuvay/DevOpsCiclo12025'
        BRANCH = 'feature/jeanProuvay'
        SONAR_HOST_URL = 'http://localhost:9000'
        SONARQUBE_SERVER = 'SonarQube-Docker' // Configurado en Jenkins > Manage Jenkins > Configure
        DOCKER_IMAGE = 'miapp-java:latest'
        DOCKER_CONTAINER = 'miapp-container'
    }
    tools {
        maven 'maven 3.9.9' // Usa el nombre que configuraste
    }

    stages {
        stage('Clonar Código') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Compilar Código') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Análisis SonarQube') {
            environment {
                // Inyectamos el secreto aquí usando su ID
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                withSonarQubeEnv('SonarQube-Docker') {
                    sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=CursoDevSecOps \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Construir Imagen Docker') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Desplegar en Docker') {
            steps {
                // Detener y eliminar contenedor previo si existe
                sh '''
                    docker rm -f ${DOCKER_CONTAINER} || true
                    docker run -d --name ${DOCKER_CONTAINER} -p 8081:8080 ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Validar que la app esté disponible') {
            steps {
                script {
                    def maxIntentos = 15
                    def intento = 1
                    def levantado = false

                    while (intento <= maxIntentos) {
                        echo "⏳ Esperando la app... intento ${intento}"
                        def status = sh(
                                script: "curl -s http://localhost:8081/",
                                returnStatus: true
                        )
                        if (status == 0) {
                            echo "✅ App disponible"
                            levantado = true
                            break
                        }
                        sleep time: 5, unit: 'SECONDS'
                        intento++
                    }

                    if (!levantado) {
                        error("❌ La app no se levantó a tiempo")
                    }
                }
            }
        }

        stage('Probar endpoint de ejemplo') {
            steps {
                echo 'Probando endpoint /api/calculadora/sumar?a=1&b=1'
                sh '''
            curl -v "http://localhost:8081/api/calculadora/sumar?a=1&b=1"
        '''
            }
        }

        stage('Ejecutar pruebas de Selenium') {
            steps {
                sh 'mvn test -Dtest=com.ejemplo.calculadora.CalculadoraUITest'
            }
        }

        stage('Mostrar logs de la app') {
            steps {
                echo 'Logs de la app (desde el contenedor):'
                sh 'docker logs miapp-container || echo "No se pudo obtener logs del contenedor"'
            }
        }

        stage('Detener la aplicación') {
            steps {
                echo 'Deteniendo el contenedor miapp-container...'
                sh 'docker stop miapp-container || echo "No se pudo detener el contenedor"'
            }
        }

        stage('Eliminar contenedor') {
            steps {
                sh 'docker rm -f miapp-container || true'
            }
        }

        stage('Eliminar imagen Docker') {
            steps {
                sh 'docker rmi -f miapp-java || true'
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminado.'
        }
        success {
            echo 'Despliegue y pruebas exitosas.'
        }
        failure {
            echo 'Algo falló durante el pipeline.'
        }
    }
}