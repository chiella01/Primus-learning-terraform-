#!/usr/bin/env groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.6'
    }
    environment {
        IMAGE_NAME = 'bemnji/demo-app:java-maven-app-1.0'
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
    }
}