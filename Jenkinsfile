pipeline {
    agent any

    parameters {
        credentials(name: 'DOCKER_CREDENTIALS', description: 'Docker Hub credentials', defaultValue: '', required: true)
        string(name: 'SONAR_URL', description: 'Sonar token', defaultValue: 'http://sonarqube:9000', required: true)
        string(name: 'SONAR_TOKEN', description: 'Sonar token', defaultValue: '', required: true)
        string(name: 'ENVIRONMENT', description: 'Environment', defaultValue: 'dev', required: true)
    }

    environment {
        PROJECT_REPO = 'https://github.com/ypillhuaman/viaandina-scheduler-msvc.git' // cambia a tu URL real
        SERVICE_NAME = 'viand-scheduler-msvc'
        IMAGE_NAME = 'yurigrow/viand-scheduler-msvc'
        IMAGE_TAG = 'latest'
        //SONAR_URL = 'http://sonarqube:9000'
        //SONAR_TOKEN = 'sqa_8b7b94ada4a8002e35b14371abc86c7dcebed6d1' // cuidado con tokens hardcodeados en producción
        DOCKER_COMPOSE_PATH = '/mnt/docker-compose'
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
                        -Dsonar.host.url=${params.SONAR_URL} \
                        -Dsonar.login=${params.SONAR_TOKEN} \
                        -DskipTests
                    """
                }
            }
        }

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
                    sh "cd ${DOCKER_COMPOSE_PATH} && APP_ENV=${params.ENVIRONMENT} docker compose pull ${SERVICE_NAME}"
                    sh "cd ${DOCKER_COMPOSE_PATH} && APP_ENV=${params.ENVIRONMENT} docker compose up -d ${SERVICE_NAME}"
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
