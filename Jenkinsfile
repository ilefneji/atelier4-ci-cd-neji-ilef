pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'ilefneji'
        REPO_NAME = 'atelier4-ci-cd-neji-ilef'
        IMAGE_SERVER = "${DOCKER_USERNAME}/${REPO_NAME}-serveur"
        IMAGE_CLIENT = "${DOCKER_USERNAME}/${REPO_NAME}-client"
    }

    stages {

        stage('Docker Build Images') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"

                    sh "docker build -t ${IMAGE_SERVER}:${imageTag} ./serveur"
                    sh "docker tag ${IMAGE_SERVER}:${imageTag} ${IMAGE_SERVER}:latest"

                    sh "docker build -t ${IMAGE_CLIENT}:${imageTag} ./client"
                    sh "docker tag ${IMAGE_CLIENT}:${imageTag} ${IMAGE_CLIENT}:latest"
                }
            }
        }

        stage('Docker Push to Registry') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh """
                          echo "${DOCKER_PASSWORD}" | docker login -u ${DOCKER_USER} --password-stdin ${DOCKER_REGISTRY}
                          docker push ${IMAGE_SERVER}:${imageTag}
                          docker push ${IMAGE_CLIENT}:${imageTag}
                          docker push ${IMAGE_SERVER}:latest
                          docker push ${IMAGE_CLIENT}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ci-cd-config/k8s-serveur-deployment.yaml"
                sh "kubectl apply -f ci-cd-config/k8s-client-deployment.yaml"
            }
        }
    }
}
