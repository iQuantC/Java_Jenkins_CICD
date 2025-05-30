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
	ACR_NAME = "javaapprepo00"
	ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
	RESOURCE_GROUP = "iquant-00"
	ACI_NAME = "java-app-container"
	ACI_REGION = "eastus"
    }
    stages {
        stage('Initialize Pipeline'){
            steps {
                echo 'Initializing Pipeline ...'
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
                echo 'Scanning File System with Trivy FS ...'
                sh "trivy fs --format table -o FSScanReport.html"
            }
        }
        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
                script {
			sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
		}
            }
        }
        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
		sh "trivy --severity HIGH,CRITICAL --cache-dir ${WORKSPACE}/.trivy-cache --no-progress --format table -o trivyFSScanReport.html image ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
	stage('Login to DockerHub'){
            steps {
                echo 'Login in to DockerHub'
		withCredentials([usernamePassword(credentialsId: 'jmDHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            		sh '''
                		echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            		'''
        	}
            }
        }
        stage('Tag & Push Docker Image'){
           steps {
               echo 'Pushing the Java App Docker Image to DockerHub'
		script {
			def imageName = "iquantc/java-app:${BUILD_NUMBER}"
			sh "docker tag java-app:${BUILD_NUMBER} ${imageName}"
			sh "docker push ${imageName}"
			sh "docker logout"
          		}
        	  }
    	     }
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
	    stage('Azure Login') {
      		steps {
        		withCredentials([file(credentialsId: 'azServicePrincipal', variable: 'AZURE_CRED')]) {
          			sh '''
            				echo "Logging into Azure CLI..."
            				az login --service-principal --username $(jq -r .clientId $AZURE_CRED) \
              					--password $(jq -r .clientSecret $AZURE_CRED) \
              					--tenant $(jq -r .tenantId $AZURE_CRED) > /dev/null
            				az account set --subscription $(jq -r .subscriptionId $AZURE_CRED)
          			'''
        			}
      			}
    		}
	    stage('Push to Azure Container Registry') {
      		steps {
			withCredentials([file(credentialsId: 'azServicePrincipal', variable: 'AZURE_CRED')]) {
        			sh '''
          				echo "Logging into ACR..."
          				az acr login --name $ACR_NAME

          				echo "Tagging Docker image..."
          				docker tag $IMAGE_NAME:$IMAGE_TAG $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG

          				echo "Pushing Docker image to ACR..."
          				docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG
        			'''
				}
      			}
    		}
	    stage('Deploy to Azure Container Instance') {
      		steps {
			withCredentials([file(credentialsId: 'azServicePrincipal', variable: 'AZURE_CRED')]) {
        			sh '''
          				echo "Deploying to Azure Container Instance...."
          				az container create \
            					--name $ACI_NAME \
            					--resource-group $RESOURCE_GROUP \
            					--image $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG \
            					--registry-login-server $ACR_LOGIN_SERVER \
            					--registry-username $(az acr credential show --name $ACR_NAME --query username -o tsv) \
            					--registry-password $(az acr credential show --name $ACR_NAME --query passwords[0].value -o tsv) \
            					--dns-name-label java-app-${BUILD_NUMBER} \
            					--ports 8090 \
            					--location $ACI_REGION \
		 				--os-type Linux \
       						--cpu 1 \
  						--memory 1.5 \
		 				--restart-policy Never

       					echo "Waiting for ACI to initialize..."
          				sleep 30

       					echo "Getting the URL of the ACI app..."
          				APP_URL=$(az container show \
            					--resource-group $RESOURCE_GROUP \
            					--name $ACI_NAME \
            					--query ipAddress.fqdn \
            					--output tsv):8090

          				echo "Application URL: http://$APP_URL"
        			'''
				}
      			}
    		}
	}
}
