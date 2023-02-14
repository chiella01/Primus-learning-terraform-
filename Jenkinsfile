#!/usr/bin/env groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.6'
    }
    environment {
        IMAGE_NAME = 'bemnji/demo-app:java-maven-app-2.0'
    }
    stages {
        stage("build app") {
            steps {
                script {
                    echo "building the applicaiton ..."
                    sh "mvn clean package"
                   
                }
            }
        }
        stage("building the image") {
            steps {
                script {
                    echo "building the image and pushing to github"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                        sh "sudo docker build -t $IMAGE_NAME ."
                        sh "echo $PASS | sudo docker login -u $USER --password-stdin"
                        sh "sudo docker push $IMAGE_NAME"

                    }
                }
            }
        }
        stage("provision server") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
                AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
            }
            steps {
                script {
                    dir('terraform') {
                        sh "terraform init"
                        sh "terraform apply -auto-approve"
                        EC2_PUBLIC_IP = sh (
                            script: "terraform output ec2_public_ip ",
                            returnStdout: true

                        ).trim()
                    }

                }
            }
        }
        stage('deploy') {
            environment {
                DOCKER_CRED = credentials('docker-hub-credentials')
            }
            steps {
                script {
                    echo "deploying docker image to EC2"
                    echo "${EC2_PUBLIC_IP}"
                    def shellCmd = "bash ./server-cmd.sh ${IMAGE_NAME} ${DOCKER_CRED_USR} ${DOCKER_CRED_PSW}"
                    def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"
                    sshagent([server-ssh-key]) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}: /home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}: /home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
    }
}