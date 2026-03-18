#!/usr/bin/env groovy

// Load your shared library from your GitHub repo
library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/Big-Kola/jenkins-shared-library.git',
   credentialsId: 'git-creds'
  ]
)

pipeline {   
    agent any

    tools {
        maven 'Maven'  // Make sure this matches your Jenkins Maven tool
    }

    environment {
        IMAGE_NAME = 'bigkola1/kola-demo-app:jma-1.1'
    }

    stages {

        stage("Build App") {
            steps {
                script {
                    echo 'Building application jar...'
                    buildJar()   // uses vars/buildJar.groovy
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    echo 'Building Docker image...'
                    buildImage(env.IMAGE_NAME)   // uses vars/buildImage.groovy
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage("Provision Server") {
            environment {
                AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
                TF_VAR_env_prefix     = 'test'
            }
            steps {
                script {
                    dir('terraform') {
                        sh "terraform init"
                        sh "terraform apply --auto-approve"

                        EC2_PUBLIC_IP = sh(
                            script: "terraform output ec2-public_ip",
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }

        stage("Deploy to EC2") {
            environment {
                DOCKER_CREDS = credentials('docker-hub-repo')
            }
            steps {
                script {
                    echo "Waiting for EC2 server to initialize..."
                    sleep(time: 90, unit: "SECONDS")

                    echo "Deploying Docker image to EC2..."
                    echo "EC2 Public IP: ${EC2_PUBLIC_IP}"

                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME} ${DOCKER_CREDS_USR} ${DOCKER_CREDS_PSW}"
                    def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"

                    sshagent(['server-ssh-key']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }

    }
}
