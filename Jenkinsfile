pipeline {
    agent any

    environment {
        AWS_REGION    = 'us-east-2'
        FRONTEND_REPO = '377946658507.dkr.ecr.us-east-2.amazonaws.com/devops-challenge-frontend'
        BACKEND_REPO  = '377946658507.dkr.ecr.us-east-2.amazonaws.com/devops-challenge-backend'
        ECS_CLUSTER   = 'devops-challenge-cluster'
        FRONTEND_SVC  = 'devops-challenge-frontend-service'
        BACKEND_SVC   = 'devops-challenge-backend-service'
    }

    stages {
        stage('Checkout code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    sh 'docker build -t frontend:latest ./frontend'
                    sh 'docker build -t backend:latest ./backend'
                }
            }
        }

        stage('Authenticate to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'was-creds']]) {
                    sh '''
                        aws --version
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 377946658507.dkr.ecr.us-east-2.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag and Push images to ECR') {
            steps {
                sh '''
                    docker tag frontend:latest $FRONTEND_REPO:latest
                    docker tag backend:latest $BACKEND_REPO:latest

                    docker push $FRONTEND_REPO:latest
                    docker push $BACKEND_REPO:latest
                '''
            }
        }

        stage('Update ECS services') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'was-creds']]) {
                    sh '''
                        aws ecs update-service --cluster $ECS_CLUSTER --service $FRONTEND_SVC --force-new-deployment --region $AWS_REGION
                        aws ecs update-service --cluster $ECS_CLUSTER --service $BACKEND_SVC --force-new-deployment --region $AWS_REGION
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}