pipeline {
    agent any

    environment {
        DOCKER_USER = "your_dockerhub_username"
        DOCKER_REGISTRY = "docker.io"
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
                sh 'docker build -t $DOCKER_USER/frontend:latest frontend'
                sh 'docker build -t $DOCKER_USER/backend:latest backend'
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        docker login -u $USER -p $PASS
                        docker push $DOCKER_USER/frontend:latest
                        docker push $DOCKER_USER/backend:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/'
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
