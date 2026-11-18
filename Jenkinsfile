pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'GitHub source checkout successful'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'sudo docker build -t my-new-app:latest .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'sudo docker rm -f new-container || true'
            }
        }

        stage('Deploy') {
            steps {
                sh 'sudo docker run -d --name new-container -p 80:80 my-new-app:latest'
            }
        }
    }
}
