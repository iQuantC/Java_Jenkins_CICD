pipeline {
    agent any
    tools {
        jdk 'java17015'
        maven 'maven387'
        //sonar 'sonar7'
    }
    environment {
        SONAR_SCANNER_HOME = tool 'sonar7'
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
        stage('Maven Build'){
            steps {
                echo 'Building Java App with Maven'
                sh 'mvn clean package'
            }
        }
        stage('JUnit Test of Java App'){
            steps {
                echo 'JUnit Testing'
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis'){
            steps {
                withCredentials([string(credentialsId: 'jmsonar', variable: 'sonarToken')]) {
                    withSonarQubeEnv('sonar') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=jm \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://172.18.0.3:9000 \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.token=$sonarToken
                        """
                    }
                }
            }
        }
        stage('Trivy FS Scan'){
            steps {
                echo 'Scanning File System with Trivy FS'
                sh "trivy fs --format table -o FSScanReport.html"
            }
        }
        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
                script {
			docker.build("java-app:${BUILD_NUMBER}")
		}
            }
        }
        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
		sh 'trivy --severity HIGH,CRITICAL --no-progress --format table -o trivyFSScanReport.html image java-app:${BUILD_NUMBER}'
            }
        }
	stage('Login to DockerHub'){
            steps {
                echo 'Login in to DockerHub'
		withCredentials([usernamePassword(
            		credentialsId: 'jmDHub',
            		usernameVariable: 'DOCKER_USER',
            		passwordVariable: 'DOCKER_PASS'
        	)]) {
            		sh '''
                		echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            		'''
        	}
            }
        }
        stage('Push Docker Image'){
            steps {
                echo 'Pushing the Java App Docker Image to DockerHub'
		//script {
			//docker.image("java-app:${BUILD_NUMBER}").push()
		//}
            }
        }
    }
}
