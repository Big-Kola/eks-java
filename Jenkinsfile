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
                // Proper way to inject AWS credentials
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'jenkins_aws_access_key_id'
                    ]
                ]) {
                    script {
                        echo "Deploying Docker image..."
                        sh '''
                        # Verify AWS credentials
                        aws sts get-caller-identity

                        # Configure kubectl for EKS
                        aws eks update-kubeconfig --region us-east-2 --name my-cluster

                        # Update deployment with new Docker image
                        kubectl set image deployment/nginx-deployment nginx=bigkola1/kola-demo-app:jma-1.1 --record
                        '''
                    }
                }
            }
        }
    }
}