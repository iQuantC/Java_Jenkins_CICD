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

On your browser, open
```sh
localhost:8080
```

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

4a9cb03b08594cff8fca741001153a5e