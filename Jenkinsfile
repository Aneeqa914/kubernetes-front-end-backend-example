pipeline {
    agent any

    environment {
        DOCKER_USER = "aneeqakamran97"
        DOCKER_REGISTRY = "docker.io"
        // AWS credentials will be picked up from Jenkins environment variables
        AWS_DEFAULT_REGION = "us-east-1"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/Aneeqa914/kubernetes-front-end-backend-example.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', 
                                                   usernameVariable: 'DOCKER_USERNAME', 
                                                   passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat "\"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe\" build --no-cache -t ${DOCKER_USER}/frontend:latest -f frontend/frontend.dockerfile frontend"
                    bat "\"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe\" build --no-cache -t ${DOCKER_USER}/backend:latest -f backend/backend.dockerfile backend"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', 
                                                   usernameVariable: 'DOCKER_USERNAME', 
                                                   passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat "\"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe\" login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                    bat "\"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe\" push ${DOCKER_USER}/frontend:latest"
                    bat "\"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe\" push ${DOCKER_USER}/backend:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
                    // AWS credentials are automatically picked up from Jenkins environment variables
                    // (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION)
                    bat 'kubectl apply -f k8s-gcp/ --validate=false'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}