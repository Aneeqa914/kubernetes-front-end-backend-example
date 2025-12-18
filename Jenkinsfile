pipeline {
    agent any

    environment {
        DOCKER_USER = "aneeqakamran97"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_CLI = "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Aneeqa914/kubernetes-front-end-backend-example.git',
                    credentialsId: 'github-token'
            }
        }
        stage('Build Docker Images') {
            steps {
                bat '"%DOCKER_CLI%" build -t %DOCKER_USER%/frontend:latest -f frontend/frontend.dockerfile frontend'
                bat '"%DOCKER_CLI%" build -t %DOCKER_USER%/backend:latest -f backend/backend.dockerfile backend'
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                        "%DOCKER_CLI%" login -u %USER% -p %PASS%
                        "%DOCKER_CLI%" push %DOCKER_USER%/frontend:latest
                        "%DOCKER_CLI%" push %DOCKER_USER%/backend:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
                    bat 'kubectl apply -f k8s/'
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
