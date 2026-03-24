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
    }

    stages {

        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    deleteDir()   // ✅ clean workspace (important)
                    git branch: "${GIT_BRANCH}",
                        credentialsId: 'naveengit',
                        url: "${GIT_REPO}"
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
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                '''
            }
        }

        // ✅ Optional Local Test
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
        }

        // ✅ FIXED: safer image update (only replace your image)
        stage('Update K8s Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        sed -i "s|naveennallamsetti/java-bank-application-project:.*|${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|" money.yml
                    '''
                }
            }
        }

        // 🔥 EKS LOGIN
        stage('Configure EKS Access') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/bin

                    echo "Connecting to EKS cluster..."

                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER

                    echo "Current Context:"
                    kubectl config current-context

                    echo "Cluster Nodes:"
                    kubectl get nodes
                '''
            }
        }

        // 🚀 Deploy
        stage('Deploy to Kubernetes') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        kubectl apply -f k8s/deployment.yml
                        kubectl apply -f k8s/service.yml
                    '''
                }
            }
        }

        // ✅ Verify
        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/java-bank-app

                    echo "Pods:"
                    kubectl get pods -o wide

                    echo "Services:"
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ FAILED"
        }
    }
}
