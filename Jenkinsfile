pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kanvii03/bluecred:latest'
        K8S_API = 'https://192.168.49.2:8443'
        KUBECONFIG = '/var/jenkins_home/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kanvii-03/bluecred.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh 'kubectl --kubeconfig=$KUBECONFIG --server=$K8S_API --insecure-skip-tls-verify=true apply -f k8s/'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl --kubeconfig=$KUBECONFIG --server=$K8S_API --insecure-skip-tls-verify=true get pods'
                sh 'kubectl --kubeconfig=$KUBECONFIG --server=$K8S_API --insecure-skip-tls-verify=true get svc'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
