pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "pavanikoduru22/nodejs-app:latest"
        DOCKERHUB_CREDENTIALS = "dockerhub-creds"
        KUBECONFIG_PATH = "/var/lib/jenkins/workspace/Jenkins_Pipeline/k8s/kubeconfig.yaml"
    }
    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Pavanikoduru/Node.js_Application.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                            docker.image("${DOCKER_IMAGE}").push()
                        }
                    }
                }
            }
        }
        stage('Load Image to Kind') {
            steps {
                sh """
                kind load docker-image ${DOCKER_IMAGE} --name jenkins-cluster
                """
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                export KUBECONFIG=${KUBECONFIG_PATH}
                kubectl apply -f k8s/k8s-deployment.yaml
                kubectl apply -f k8s/k8s-service.yaml
                kubectl get pods
                kubectl get svc
                """
            }
        }
    }
}
