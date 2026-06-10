pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'romergi1/colombia-departamentos-api'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Clonar repositorio') {
            steps {
                checkout scm
            }
        }

        stage('Instalar dependencias') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Ejecutar pruebas') {
            steps {
                sh 'npm test'
            }
        }

        stage('Análisis estático') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Construir imagen Docker') {
            steps {
                sh """
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    -t ${DOCKER_IMAGE}:latest .
                """
            }
        }

        stage('Autenticarse en DockerHub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login \
                    -u "$DOCKERHUB_CREDENTIALS_USR" \
                    --password-stdin
                '''
            }
        }

        stage('Publicar imagen en DockerHub') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo 'Pipeline CD ejecutado exitosamente.'
        }

        failure {
            echo 'Pipeline CD falló. Revisar logs de Jenkins.'
        }
    }
}