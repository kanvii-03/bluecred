pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        BACKEND_IMAGE = 'kanvii03/bluecarbon-backend'
        FRONTEND_IMAGE = 'kanvii03/bluecarbon-frontend'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Backend') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE/backend:/app" \
                      -w /app node:22-alpine \
                      sh -c "npm ci && node --check src/index.js"
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE/frontend:/app" \
                      -w /app node:22-alpine \
                      sh -c "npm ci && npm run build"
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                docker build -f backend/dockerfile -t "$BACKEND_IMAGE:$BUILD_NUMBER" -t "$BACKEND_IMAGE:latest" ./backend
                docker build -f frontend/dockerfile -t "$FRONTEND_IMAGE:$BUILD_NUMBER" -t "$FRONTEND_IMAGE:latest" ./frontend
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
                        docker push "$BACKEND_IMAGE:$BUILD_NUMBER"
                        docker push "$BACKEND_IMAGE:latest"
                        docker push "$FRONTEND_IMAGE:$BUILD_NUMBER"
                        docker push "$FRONTEND_IMAGE:latest"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/
                    kubectl set image deployment/bluecarbon-backend backend="$BACKEND_IMAGE:$BUILD_NUMBER"
                    kubectl set image deployment/bluecarbon-frontend frontend="$FRONTEND_IMAGE:$BUILD_NUMBER"
                    kubectl rollout status deployment/bluecarbon-backend --timeout=180s
                    kubectl rollout status deployment/bluecarbon-frontend --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'BlueCred CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output for the first error.'
        }
    }
}
