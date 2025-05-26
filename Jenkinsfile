pipeline {
    agent any
    tools {
        jdk 'java17015'
        maven 'maven387'
    }
    stages {
        stage('Initialize Pipeline'){
            steps {
                echo 'Initializing Pipeline'
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        stage('Checkout GitHub Codes'){
            steps {
                echo 'Checking out GitHub Codes'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jm', url: 'https://github.com/iQuantC/Java_Jenkins_CICD.git']])
            }
        }
        stage('Maven Build & Test'){
            steps {
                echo 'Building Java App with Maven'
                sh 'mvn clean package && mvn test'
            }
        }
        stage('SonarQube Analysis'){
            steps {
                echo 'Static Code Analysis with SonarQube'
            }
        }
        stage('Quality Gate'){
            steps {
                echo 'Check Quality Gate'
            }
        }
        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
            }
        }
        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
            }
        }
        stage('Push Docker Image'){
            steps {
                echo 'Pushing the Java App Docker Image to DockerHub'
            }
        }
    }
}
