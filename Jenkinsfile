pipeline {
    agent any
    tools {

    }
    environment {

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
            }
        }
        stage('Maven Build & Test'){
            steps {
                echo 'Building Java App with Maven'
            }
        }
        stage('Maven Package'){
            steps {
                echo 'Packaging artifact with Maven'
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
        post {
            always {
                cleanWs()
            }
            success {
                echo 'Pipeline Completed Successfully!'
            }
            failure {
                echo 'Pipeline Failed!'
            }
        }
    }
}