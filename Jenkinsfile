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
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
                AWS_DEFAULT_REGION = 'us-east-2' // your AWS region
                IMAGE_NAME = 'bigkola1/kola-demo-app:jma-1.1' // Docker image to deploy
            }
            steps {
                script {
                   echo 'deploying docker image...'
                   sh '''
                   # Configure kubectl with EKS
                   aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name my-cluster
                   
                   # Deploy/update deployment in Kubernetes
                   kubectl set image deployment/nginx-deployment nginx=${IMAGE_NAME} --record
                   '''
                }
            }
        }
    }
}