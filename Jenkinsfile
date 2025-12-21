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
                withCredentials([
                    file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG'),
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    bat '''
                        @echo off
                        REM Create AWS credentials directory
                        if not exist "%USERPROFILE%\.aws" mkdir "%USERPROFILE%\.aws"
                        
                        REM Write AWS credentials to file
                        echo [default] > "%USERPROFILE%\.aws\credentials"
                        echo aws_access_key_id=%AWS_ACCESS_KEY_ID% >> "%USERPROFILE%\.aws\credentials"
                        echo aws_secret_access_key=%AWS_SECRET_ACCESS_KEY% >> "%USERPROFILE%\.aws\credentials"
                        
                        REM Write AWS config
                        echo [default] > "%USERPROFILE%\.aws\config"
                        echo region=us-east-1 >> "%USERPROFILE%\.aws\config"
                        
                        REM Deploy to Kubernetes
                        kubectl apply -f k8s-gcp/ --validate=false
                        
                        REM Clean up credentials
                        del "%USERPROFILE%\.aws\credentials"
                        del "%USERPROFILE%\.aws\config"
                    '''
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
