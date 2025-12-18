pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'ilefneji'
        REPO_NAME = 'atelier4-ci-cd-neji-ilef'

        IMAGE_SERVER = "${DOCKER_USERNAME}/${REPO_NAME}-serveur"
        IMAGE_CLIENT = "${DOCKER_USERNAME}/${REPO_NAME}-client"

        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Docker Build Images') {
            steps {
                sh '''
                  echo "Building images with tag: ${IMAGE_TAG}"

                  docker build -t ${IMAGE_SERVER}:${IMAGE_TAG} ./serveur
                  docker tag ${IMAGE_SERVER}:${IMAGE_TAG} ${IMAGE_SERVER}:latest

                  docker build -t ${IMAGE_CLIENT}:${IMAGE_TAG} ./client
                  docker tag ${IMAGE_CLIENT}:${IMAGE_TAG} ${IMAGE_CLIENT}:latest
                '''
            }
        }

        stage('Docker Push to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                      echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin docker.io

                      docker push ${IMAGE_SERVER}:${IMAGE_TAG}
                      docker push ${IMAGE_CLIENT}:${IMAGE_TAG}

                      docker push ${IMAGE_SERVER}:latest
                      docker push ${IMAGE_CLIENT}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  echo "Current directory:"
                  pwd
                  echo "Listing repo files:"
                  ls -l
                  echo "Listing ci-cd-config:"
                  ls -l ci-cd-config

                  kubectl apply -f ci-cd-config/k8s-serveur-deployment.yaml
                  kubectl apply -f ci-cd-config/k8s-client-deployment.yaml
                '''
            }
        }
    }
}
