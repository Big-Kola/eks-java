#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                }
            }
        }
        stage('deploy') {
            steps {
                withEnv([
                    "AWS_ACCESS_KEY_ID=${credentials('jenkins_aws_access_key_id')}",
                    "AWS_SECRET_ACCESS_KEY=${credentials('jenkins-aws_secret_access_key')}",
                    "AWS_DEFAULT_REGION=us-east-2",
                    "IMAGE_NAME=bigkola1/kola-demo-app:jma-1.1"
                ]) {
                    script {
                        echo 'Deploying Docker image...'
                        sh '''
                        # Verify AWS credentials
                        aws sts get-caller-identity
                        
                        # Configure kubectl for EKS
                        aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name my-cluster
                        
                        # Update deployment with new Docker image
                        kubectl set image deployment/nginx-deployment nginx=${IMAGE_NAME} --record
                        '''
                    }
                }
            }
        }
    }
}