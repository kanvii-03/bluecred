pipeline {
    agent any

    environment {
        BACKEND_IMAGE = 'kanvii03/bluecarbon-backend:latest'
        FRONTEND_IMAGE = 'kanvii03/bluecarbon-frontend:latest'
        KUBECONFIG = '/var/jenkins_home/.kube/config'
        K8S_API = 'https://192.168.49.2:8443'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kanvii-03/bluecred.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t $BACKEND_IMAGE -f backend/dockerfile ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t $FRONTEND_IMAGE -f frontend/dockerfile ./frontend'
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$BACKEND_IMAGE"
                        docker push "$FRONTEND_IMAGE"
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    kubectl --kubeconfig="$KUBECONFIG" --server="$K8S_API" --insecure-skip-tls-verify=true apply -f k8s/
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl --kubeconfig="$KUBECONFIG" --server="$K8S_API" --insecure-skip-tls-verify=true get pods
                    kubectl --kubeconfig="$KUBECONFIG" --server="$K8S_API" --insecure-skip-tls-verify=true get services
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
