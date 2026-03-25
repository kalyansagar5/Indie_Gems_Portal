pipeline {
agent any

environment {
    WORK_DIR = "/var/lib/jenkins/workspace/Game"
    IMAGE_NAME = "indie-gems"
    IMAGE_TAG = "${BUILD_NUMBER}"
    CONTAINER_NAME = "indie-gems-container"
    PORT = "9676"
    DOCKERHUB_USER = "kalyansagar5"
    DOCKER_CREDS = "docker_cred"
    CONTAINER_PORT = "80"
    AWS_REGION = "us-east-1"
    EKS_CLUSTER = "mycluster"
    KUBECONFIG = "/var/lib/jenkins/.kube/config"
    EMAIL_TO = "ravee2288@gmail.com"
}

stages {

    stage('Checkout Code') {
        steps {
            dir("${WORK_DIR}") {
                git branch: 'main', url: 'https://github.com/kalyansagar5/Indie_Gems_Portal.git'
            }
        }
        post {
            success {
                emailext subject: "Checkout SUCCESS - Build ${BUILD_NUMBER}",
                         body: "Code checkout completed successfully.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Checkout FAILED - Build ${BUILD_NUMBER}",
                         body: "Checkout stage failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }

    stage('Build Docker Image') {
        steps {
            dir("${WORK_DIR}") {
                sh '''
                    docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }
        post {
            success {
                emailext subject: "Docker Build SUCCESS - Build ${BUILD_NUMBER}",
                         body: "Docker image built successfully.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Docker Build FAILED - Build ${BUILD_NUMBER}",
                         body: "Docker build failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }

    stage('DockerHub Login') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: "${DOCKER_CREDS}",
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                """
            }
        }
        post {
            success {
                emailext subject: "Docker Login SUCCESS - Build ${BUILD_NUMBER}",
                         body: "DockerHub login successful.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Docker Login FAILED - Build ${BUILD_NUMBER}",
                         body: "DockerHub login failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }

    stage('Push Image to DockerHub') {
        steps {
            sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
        post {
            success {
                emailext subject: "Docker Push SUCCESS - Build ${BUILD_NUMBER}",
                         body: "Image pushed to DockerHub.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Docker Push FAILED - Build ${BUILD_NUMBER}",
                         body: "Docker push failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            sh '''
                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
            '''
        }
        post {
            success {
                emailext subject: "Deployment SUCCESS - Build ${BUILD_NUMBER}",
                         body: "Kubernetes deployment successful.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Deployment FAILED - Build ${BUILD_NUMBER}",
                         body: "Deployment failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
                kubectl rollout status deployment python-devops-app || true
                kubectl get pods -o wide
                kubectl get svc
            '''
        }
        post {
            success {
                emailext subject: "Verification SUCCESS - Build ${BUILD_NUMBER}",
                         body: "Deployment verified successfully.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
            failure {
                emailext subject: "Verification FAILED - Build ${BUILD_NUMBER}",
                         body: "Verification failed.\n${BUILD_URL}",
                         to: "${EMAIL_TO}"
            }
        }
    }
}

post {
    always {
        emailext subject: "Stage Complete - Build ${BUILD_NUMBER}",
                 body: "Stage completed (success or fail).\n${BUILD_URL}",
                 to: "${env.EMAIL_TO}"
    }
}

}
