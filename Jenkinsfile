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
                    docker run -d --name ${DOCKER_CONTAINER} -p 8081:8081 ${DOCKER_IMAGE}
                '''
            }
        }
/*
        stage('Test de UI con Selenium') {
            steps {
                // Asegúrate de que el contenedor de Selenium esté en red accesible
                sh '''
                    docker run --rm --network host \
                        -v $(pwd)/tests:/tests \
                        selenium/standalone-chrome:latest \
                        sh -c "cd /tests && ./run-selenium-tests.sh"
                '''
            }
        }

 */
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