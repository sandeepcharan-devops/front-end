pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = "vipparlasandeepcharan/front-end"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image \
                      --severity CRITICAL \
                      --exit-code 1 \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo ${DOCKERHUB_CREDS_PSW} | docker login \
                      -u ${DOCKERHUB_CREDS_USR} \
                      --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed — image not pushed'
        }
        success {
            echo "Image ${IMAGE_NAME}:${IMAGE_TAG} pushed successfully"
        }
    }
}
