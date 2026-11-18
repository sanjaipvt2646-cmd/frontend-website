
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION     = 'eu-north-1'
        AWS_ACCOUNT_ID = '887074880802'
        ECR_REPOSITORY = 'frontend-website'

        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'frontend-container'
        ECR_REGISTRY   = '887074880802.dkr.ecr.eu-north-1.amazonaws.com'
        ECR_URI        = '887074880802.dkr.ecr.eu-north-1.amazonaws.com/frontend-website'
    }

    stages {

        stage('1. Checkout GitHub') {
            steps {
                echo 'Pulling source code from GitHub'
                checkout scm
                sh 'ls -la'
            }
        }

        stage('2. Verify AWS Identity') {
            steps {
                sh '''
                    aws sts get-caller-identity
                '''
            }
        }

        stage('3. ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('4. Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${ECR_URI}:${IMAGE_TAG} .
                '''
            }
        }

        stage('5. Tag Docker Image') {
            steps {
                sh '''
                    docker tag \
                    ${ECR_URI}:${IMAGE_TAG} \
                    ${ECR_URI}:latest
                '''
            }
        }

        stage('6. Push Image to ECR') {
            steps {
                sh '''
                    docker push ${ECR_URI}:${IMAGE_TAG}
                    docker push ${ECR_URI}:latest
                '''
            }
        }

        stage('7. Pull Image from ECR') {
            steps {
                sh '''
                    docker pull ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('8. Stop Old Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('9. Deploy Container') {
            steps {
                sh '''
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    --restart unless-stopped \
                    -p 80:80 \
                    ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('10. Verify Deployment') {
            steps {
                sh '''
                    docker ps
                    curl -f http://localhost/
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Image pushed to ECR and application deployed!'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output.'
        }

        always {
            echo 'ECR CI/CD Pipeline execution completed.'
        }
    }
}
