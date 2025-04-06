pipeline {
    agent any

    parameters {
        credentials(name: 'DOCKER_CREDENTIALS', description: 'Docker Hub credentials', defaultValue: '', required: true)
    }

    environment {
        PROJECT_REPO = 'https://github.com/ypillhuaman/viaandina-scheduler-msvc.git' // cambia a tu URL real
        IMAGE_NAME = 'yurigrow/viand-scheduler-msvc'
        IMAGE_TAG = 'latest'
        SONAR_URL = 'http://sonarqube:9000'
        SONAR_TOKEN = 'sqa_c108b77cfc230481ad47558186e1b0979c9ff7f8' // cuidado con tokens hardcodeados en producción
        DOCKER_COMPOSE_PATH = '/home/fortuna/viaandina/docker-compose/docker-compose.yml'
    }

    stages {
        stage('Clonar Repositorio') {
            steps {
                dir('project') {
                    git branch: 'master', url: "${env.PROJECT_REPO}"
                }
            }
        }

        stage('Análisis con SonarQube') {
            steps {
                dir('project') {
                    sh 'chmod +x ./mvnw'
                    sh """
                        ./mvnw clean verify sonar:sonar \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -DskipTests
                    """
                }
            }
        }

        // No se puede usar waitForQualityGate aquí porque no usamos el plugin con withSonarQubeEnv

        stage('Build Docker Image') {
            steps {
                dir('project') {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', "${params.DOCKER_CREDENTIALS}") {
                            def app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                            app.push()
                        }
                    }
                }
            }
        }

        stage('Actualizar Docker Compose') {
            steps {
                script {
                    sh "docker compose -f ${DOCKER_COMPOSE_PATH} pull ${IMAGE_NAME}"
                    sh "docker compose -f ${DOCKER_COMPOSE_PATH} up -d ${IMAGE_NAME}"
                }
            }
        }
    }

    post {
        failure {
            echo "El pipeline falló. Revisa los logs para más detalles."
        }
    }
}
