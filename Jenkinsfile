
pipeline {
    agent any

    environment {
        BLUE_IMAGE = 'vle8-blue:v1'
        GREEN_IMAGE = 'vle8-green:v2'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'VLE 8 source code ready'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t vle8-blue:v1 ./blue'
                sh 'docker build -t vle8-green:v2 ./green'
            }
        }

        stage('Load Images into Minikube') {
            steps {
                sh 'docker save vle8-blue:v1 -o blue.tar'
                sh 'docker exec -i minikube ctr -n k8s.io images import - < blue.tar'
                sh 'docker save vle8-green:v2 -o green.tar'
                sh 'docker exec -i minikube ctr -n k8s.io images import - < green.tar'
            }
        }

        stage('Deploy Blue and Green') {
            steps {
                sh 'kubectl apply -f k8s/blue-deployment.yaml'
                sh 'kubectl apply -f k8s/green-deployment.yaml'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh 'kubectl patch service vle8-service --type=merge -p \'{"spec":{"selector":{"app":"vle8-green"}}}\''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl rollout status deployment/vle8-blue --timeout=120s'
                sh 'kubectl rollout status deployment/vle8-green --timeout=120s'
                sh 'kubectl get deployments,pods,services'
            }
        }
    }

    post {
        success {
            echo 'VLE 8 BLUE-GREEN DEPLOYMENT SUCCESSFUL!'
        }
        failure {
            echo 'VLE 8 DEPLOYMENT FAILED!'
        }
    }
}
