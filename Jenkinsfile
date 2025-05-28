pipeline {
    agent any
    tools {
        jdk 'java17015'
        maven 'maven387'
    }
    environment {
        SONAR_SCANNER_HOME = tool 'sonar7'
	PROJECT_ID = "focal-dock-440200-u5"
        IMAGE_NAME = "java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE_NAME = "us-docker.pkg.dev/${PROJECT_ID}/java-app-repo-02/${IMAGE_NAME}:${IMAGE_TAG}"
	SERVICE_NAME = "java-app-service"
	REGION = "us-central1"
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
			sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
			//docker.build("java-app:${BUILD_NUMBER}")
		}
            }
        }
        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
		//sh "trivy --severity HIGH,CRITICAL --no-progress --format table -o trivyFSScanReport.html image ${IMAGE_NAME}:${IMAGE_TAG}"
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
        //stage('Tag & Push Docker Image'){
        //    steps {
        //        echo 'Pushing the Java App Docker Image to DockerHub'
	//	script {
	//		def imageName = "iquantc/java-app:${BUILD_NUMBER}"
	//		sh "docker tag java-app:${BUILD_NUMBER} ${imageName}"
	//		sh "docker push ${imageName}"
	//		sh "docker logout"
        //   		}
        //	  }
    	//      }
	stage('Authenticate with GCP & Push to Artifact Registry') {
            steps {
		withCredentials([file(credentialsId: 'gcp-jmsa', variable: 'gcpCred')]) {
			withEnv(["GOOGLE_APPLICATION_CREDENTIALS=$gcpCred"]) {
		    		sh '''
                    			echo Activating GCP service account...
                    			gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    			gcloud config set project focal-dock-440200-u5
                    			echo Configuring Docker to use gcloud credentials...
                    			gcloud auth configure-docker us-docker.pkg.dev --quiet
                		'''
		    		script {
			//		sh '''
			//		if ! gcloud artifacts repositories create java-app-repo --repository-format=docker --location=us --description="Docker repository" --project=focal-dock-440200-u5 >/dev/null 2>&1; then 
			//			echo "Creating Artifact Registry repository..."
    			//			gcloud artifacts repositories create java-app-repo \
      			//				--repository-format=docker \
      			//				--location=us \
      			//				--description="Docker repository" \
      			//				--project=focal-dock-440200-u5
  			//		else
    			//			echo "Artifact Registry repository already exists. Skipping..."
  			//		fi
     			//		'''
					sh '''
						gcloud artifacts repositories create java-app-repo-02 --repository-format=docker --location=us --description="Docker repository" --project=focal-dock-440200-u5
      					'''
					sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}"
					sh "docker push ${FULL_IMAGE_NAME}"
					echo "Image pushed to: ${FULL_IMAGE_NAME}"
						}
					}
				}
            		}
        	}
	    stage('Deploy to Cloud Run') {
		    steps {
			withCredentials([file(credentialsId: 'gcp-jmsa', variable: 'gcpCred')]) {
			   withEnv(["GOOGLE_APPLICATION_CREDENTIALS=$gcpCred"]) {
		    		sh '''
                    			echo "Deploying $FULL_IMAGE_NAME to Cloud Run..."
          				gcloud run deploy $SERVICE_NAME \
            					--image=$FULL_IMAGE_NAME \
            					--region=$REGION \
            					--platform=managed \
            					--allow-unauthenticated \
		 				--port=8090 \
            					--memory=512Mi \
            					--quiet
                		'''
			   	}
			   }
		    }
	      }
	    stage('Get Cloud Run Service URL') {
            	steps {
			withCredentials([file(credentialsId: 'gcp-jmsa', variable: 'gcpCred')]) {
			   	withEnv(["GOOGLE_APPLICATION_CREDENTIALS=\${gcpCred}"]) {
                			sh '''
		   				echo "Getting Cloud Run service URL..."
                    				SERVICE_URL=$(gcloud run services describe $SERVICE_NAME \
                        				--platform managed \
                        				--region $REGION \
                        				--format="value(status.url)")
			    			echo "Service deployed successfully!"
                        			echo "Service URL: $SERVICE_URL"
                			'''
					}
				}
            		}
        	}
	}
}
