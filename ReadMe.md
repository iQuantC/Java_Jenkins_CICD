# Jenkins CI/CD Pipeline for Java Application


## Requirements
1. Java:        Language & Framework 
2. Spring Boot: Creates Web Applications in Java
2. Maven:       Java Application Build Tool
3. Shell:       Commands to Build, Test, and Deploy
3. Jenkins:     CI/CD Pipeline Tool
4. Docker:      Containerization Tool
5. 

### Set up Environment

Install Java
```sh
sudo apt update
sudo apt install openjdk-11-jre-headless
java -version
```

Install Maven
```sh
sudo apt update
sudo apt install maven
mvn -version
```

### Build the Application
```sh
mvn clean package
```
This creates 2 JAR files in the target directory:
1. hello-world-1.0-SNAPSHOT.jar - Executable JAR file with all dependencies included
2. hello-world-1.0-SNAPSHOT.jar.original - Original Non-executable JAR without dependencies (back up of JAR file
                                            before Spring Boot processed it)



### Run the Tests
```sh
mvn test
```

### Run The App Locally
Run this command below in a dedicated terminal:
```sh
java -jar target/hello-world-1.0-SNAPSHOT.jar
```

Open your browser to: 
```sh
http://localhost:8080
```
Ctrl + C to exit

### Additional Configuration (Optional)
If you want to change the port from the default 8080, create a file - src/main/resources/application.properties
with the following content:
```sh
server.port=8090
```

To run the Java App Locally, use the command:
```sh
java -jar target/hello-world-1.0-SNAPSHOT.jar --server.port=8090
```


## Dockerize the Java App

### Build the Docker image
```sh
docker images
```

I'm gonna use port 8090
```sh
docker build -t java-app .
```

### Run the Docker container
```sh
docker run -p 8090:8090 java-app
```

Open your browser to: 
```sh
http://localhost:8080
```
Ctrl + C to exit


## Set up Jenkins with a Container
We want to build a docker image later with our Jenkins Pipeline, so we need a Jenkins Container or VM with Docker in it (i.e., Docker-in-Docker or DinD)

### Create a Docker Network
```sh
docker network ls
```
```sh
docker network create jenkins-java
```

```sh
docker run -d --name jenkins-dind \
-p 8080:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
-u root \
-e DOCKER_GID=$(getent group docker | cut -d: -f3) \
--network jenkins-java \
jenkins/jenkins:lts
```

### Check the Docker Processes Running
```sh
docker ps
```
Get Jenkins Container IP Address
```sh
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins-dind
```
On your browser, open
```sh
ContainerIP:8080
```

Note: If you see the error "It appears that your reverse proxy set up is broken.", go to Manage Jenkins > System and change the Jenkins URL from localhost:8080 to ContainerIP:8080.


### Log into the Jenkins Container
```sh
docker exec -it jenkins-dind bash
```

### Set up Docker in the Jenkins container
Inside the container, run the following commands: 
```sh
docker --version
```

```sh
groupadd -for -g $DOCKER_GID docker
usermod -aG docker jenkins
exit
```

```sh
docker ps
```

```sh
docker restart jenkins-dind
```

### Check the Logs of the Jenkins Container
This will also provide you the Jenkins Initial Password. Use the password to login to Jenkins & Install suggested plugins
```sh
docker logs -f jenkins-dind
```
Ctrl + C to exit logs

### Integrate Jenkins & GitHub Repository
In your GitHub Account, 
1. Create a PAT  with scope (repo & admin:repo_hook), and 
2. Add it as a Credential (Kind: Username with Psswd) on Jenkins UI
    1. Username:    iQuantC
    2. Password:    PAT here
3. Create Pipeline Job with Definition: "Pipeline script from SCM", and complete the fields SCM, Repository URL, Credentials, Branch Specifier, and Script Path. 
4. Apply and Save when done.


### Install Java & Maven in the Jenkins Container
```sh
docker exec -it jenkins-dind bash
```

Check Java version (Usually installed with Jenkins since Java is one of Jenkins' Dependencies) & JAVA_HOME
```sh
java -version
```
```sh
mvn -version
```

Install Maven & get MAVEN_HOME as well (usually: /usr/share/maven)
```sh
apt update -y
apt install maven -y
mvn -version
exit
```
***In the Jenkins GUI***
1. Install Maven Integration Plugin 
2. Set up Maven installation in Manage Jenkins > Tools > Maven installations - Uncheck "Install Automatically"
    1. Name:        maven387
    2. MAVEN_HOME:  /usr/share/maven
3. Set up JDK installation in Manage Jenkins > Tools > JDK installations - Uncheck "Install Automatically"
    1. Name:        java17015
    2. JAVA_HOME:   /opt/java/openjdk
4. Add these tools in the Jenkinsfile for 'jdk' and 'maven'.



## Set up SonarQube Analysis with a Container

Run the command below to create the SonarQube Container on the same Docker network as Jenkins:
```sh
docker run -d --name sonarqube-dind \
-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
-p 9000:9000 \
--network jenkins-java \
sonarqube:latest
```

Check the logs of the SonarQube Container (Wait till SonarQube is Operational):
```sh
docker logs -f sonarqube-dind
```

### Check the Docker Processes Running
```sh
docker ps
```
Get SonarQube Container IP Address
```sh
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' sonarqube-dind
```
On your browser, open
```sh
ContainerIP:9000
```

Login to SonarQube GUI with 
```sh
Username: admin
Password: admin
```
and Change the password.


### Create Project in SonarQube

Create a Local Project in SonarQube and provide the ff:
1. Project display name, 
2. Project key, 
3. GitHub branch (main), 
4. Use the global setting, and 
5. Create project. 


### Create a SonarQube Token and Add it to Jenkins Credentials:

1. Click on the User Account, and click on "My Account"
2. Go to Security, create a token of type "Global Analysis Token", expiry date, and Generate.
3. Copy and Save the token somewhere safe.
4. Go to Jenkins Credentials, select Kind "Secret text", Paste sonar token as the secret and provide ID and description.
5. Install the "SonarQube Scanner" and "Sonar Quality Gates" Plugins.
6. Go to Jenkins > Manage Jenkins > Systems > SonarQube Installation, Name: sonar, SonarQube URL (http://ContainerIP:9000), and Server auth token: select the credential. Apply and Save.
7. Go to Jenkins > Manage Jenkins > Tools > Add Name: sonar7, Install Automatically. Save and Apply


