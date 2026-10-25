pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/minterjt'
        IMAGE_TAG = "lab39-build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/srxxm49mps-png/225-lab3-9.git'
        KUBECONFIG = credentials('minterjt')
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'test "$(kubectl get nodes -o jsonpath=\'{.items[0].metadata.name}\')" = "minterjt-fl26-pool1-sfjhj-t2bf2"'
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry-1.docker.io', "${DOCKER_CREDENTIALS_ID}") {
                        docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                sh 'kubectl apply -f metallb-production.yaml'
                sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                sh 'kubectl apply -f deployment-dev.yaml'
                sh 'kubectl rollout status deployment/minterjt-lab39-flask --timeout=300s'
                sh "kubectl wait --for=jsonpath='{.status.loadBalancer.ingress[0].ip}'=10.48.229.34 service/minterjt-lab39-flask-service --timeout=120s"
            }
        }

        stage('Verify Flask App') {
            steps {
                sh 'curl --fail --retry 10 --retry-delay 3 --retry-connrefused http://10.48.229.34:5000/ | grep "Add Contact"'
                sh 'kubectl get deployment minterjt-lab39-flask'
                sh 'kubectl get service minterjt-lab39-flask-service'
            }
        }
    }
}
