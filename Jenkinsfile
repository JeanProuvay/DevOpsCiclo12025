pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/JeanProuvay/DevOpsCiclo12025'
        BRANCH = 'feature/jeanProuvay'
        SONARQUBE_SERVER = 'SonarScanner' // Configurado en Jenkins > Manage Jenkins > Configure
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
                sh 'mvn clean package -DskipTests' // O usa "mvn clean package"
            }
        }

        stage('Análisis SonarQube') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh './mvn sonar:sonar'
                }
            }
        }
/*
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
                    docker run -d --name ${DOCKER_CONTAINER} -p 8080:8080 ${DOCKER_IMAGE}
                '''
            }
        }

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