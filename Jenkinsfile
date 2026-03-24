pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/bankapp"
        GIT_REPO   = "https://github.com/naveennallamsetti/Java-Bank-Application-Project.git"
        GIT_BRANCH = "main"

        DOCKERHUB_USER = "naveennallamsetti"
        IMAGE_NAME     = "java-bank-application-project"
        IMAGE_TAG      = "${BUILD_NUMBER}"

        DOCKER_CREDS   = "naveendocker"

        AWS_REGION = "us-east-1"
        EKS_CLUSTER = "for-tasks"

        CONTAINER_NAME = "bank-app-container"
        HOST_PORT      = "8081"
        CONTAINER_PORT = "8080"
        NOTIFY_EMAIL = "naveennallametti60@gmail.com"
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    deleteDir()
                    git branch: "${GIT_BRANCH}",
                        credentialsId: 'naveengit',
                        url: "${GIT_REPO}"
                }
            }
            post {
                success {
                    emailext(
                        subject: "✅ Checkout Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Checkout of repository ${GIT_REPO} succeeded.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Checkout Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Checkout of repository ${GIT_REPO} failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                        docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    '''
                }
            }
            post {
                success {
                    emailext(
                        subject: "✅ Docker Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Docker image ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} built successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Docker Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Docker build failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
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
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
            post {
                success {
                    emailext(
                        subject: "✅ DockerHub Login Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Logged in to DockerHub successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ DockerHub Login Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "DockerHub login failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                '''
            }
            post {
                success {
                    emailext(
                        subject: "✅ Docker Push Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Docker image pushed to DockerHub successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Docker Push Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Docker push failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Deploy Container (Local Test)') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} \
                    --name ${CONTAINER_NAME} \
                    ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
            post {
                success {
                    emailext(
                        subject: "✅ Local Container Deploy Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Docker container deployed locally successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Local Container Deploy Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Local container deployment failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        // stage('Update K8s Image') {
        //     steps {
        //         dir("${WORK_DIR}") {
        //             sh '''
        //                 sed -i "s|image:.*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|" money.yml
        //             '''
        //         }
        //     }
        stage('Update K8s Image') {
    steps {
        sh """
            kubectl set image deployment/java-bank-app \
            java-bank-container=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} \
            --record
        """
    }
}
            post {
                success {
                    emailext(
                        subject: "✅ K8s YAML Updated: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Kubernetes deployment YAML updated successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ K8s YAML Update Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Failed to update Kubernetes YAML. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/bin
                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                    kubectl config current-context
                    kubectl get nodes
                '''
            }
            post {
                success {
                    emailext(
                        subject: "✅ EKS Configured Successfully: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Connected to EKS cluster successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ EKS Configuration Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Failed to connect to EKS cluster. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        kubectl apply -f money.yml
                    '''
                }
            }
            post {
                success {
                    emailext(
                        subject: "✅ K8s Deployment Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Application deployed to Kubernetes successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ K8s Deployment Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Kubernetes deployment failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/java-bank-app || true
                    kubectl get pods -o wide
                    kubectl get svc
                    kubectl get ingress
                '''
            }
            post {
                success {
                    emailext(
                        subject: "✅ Deployment Verified: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Deployment verification completed successfully.",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Deployment Verification Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Deployment verification failed. Check build URL: ${env.BUILD_URL}",
                        to: "${NOTIFY_EMAIL}"
                    )
                }
            }
        }
    }
}
