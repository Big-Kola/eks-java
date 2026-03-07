#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        // AWS credentials stored in Jenkins
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
        AWS_DEFAULT_REGION = 'us-east-2'
        
        // Docker Hub credentials
        DOCKERHUB_CREDENTIALS = 'docker-hub-repo'
        IMAGE_NAME = 'bigkola1/kola-demo-app:jma-1.1'
    }
    stages {
        stage('Build App') {
            steps {
                script {
                    echo "Building the application..."
                    // Replace with your actual build commands
                    // e.g., sh 'npm install && npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Logging in to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying Docker image to EKS..."
                    sh '''
                    aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name my-cluster
                    kubectl set image deployment/nginx-deployment nginx=${IMAGE_NAME} --record
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}