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

	stage('Update GitOps Repo') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'github-creds',
            usernameVariable: 'GIT_USER',
            passwordVariable: 'GIT_TOKEN'
        )]) {
            sh """
    # Remove if exists from previous build
    rm -rf sockshop-gitops

    # Clone GitOps repo
    git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/sandeepcharan-devops/sockshop-gitops.git
    cd sockshop-gitops

    # Update image tag
    sed -i 's|image: vipparlasandeepcharan/front-end:.*|image: vipparlasandeepcharan/front-end:${IMAGE_TAG}|g' manifests/front-end/deployment.yaml

    # Commit and push
    git config user.email "jenkins@sockshop.com"
    git config user.name "Jenkins"
    git add manifests/front-end/deployment.yaml
    git commit -m "ci: update front-end image to ${IMAGE_TAG}"
    git push origin main
"""
        }
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
