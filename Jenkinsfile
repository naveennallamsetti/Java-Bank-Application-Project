pipeline {
    agent any

    environment {
        GIT_REPO   = "https://github.com/naveennallamsetti/Java-Bank-Application-Project.git"
        GIT_BRANCH = "main"

        DOCKERHUB_USER = "naveennallamsetti"
        IMAGE_NAME     = "java-bank-application-project"
        IMAGE_TAG      = "${BUILD_NUMBER}"

        DOCKER_CREDS   = "naveendocker"

        CONTAINER_NAME = "bank-app-container"
        HOST_PORT      = "8081"
        CONTAINER_PORT = "8080"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}",
                    credentialsId: 'naveengit',
                    url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest .
                """
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
        }

        stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy Container (Optional Local Test)') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d \
                -p ${HOST_PORT}:${CONTAINER_PORT} \
                --name ${CONTAINER_NAME} \
                ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
            export KUBECONFIG=$KUBECONFIG

            kubectl get nodes

            kubectl set image deployment/java-bank-app \
            java-bank-container=''' + "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}" + '''
            '''
        }
    }
}

        // ✅ OPTIONAL: Deploy to Kubernetes (EKS)
        // stage('Deploy to Kubernetes') {
        //     steps {
        //         sh """
        //         kubectl set image deployment/java-bank-app \
        //         java-bank-container=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
        //         """
        //     }
        // }
    }
}
