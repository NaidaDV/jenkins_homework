# Second homework - container with nginx inside
## Table of contents 
1) App source 
2) Jenkinsfile - content with comments
3) Dockerfile - content with comments
4) Workflow
## App source
For this homework I choose the Java project, stored in https://github.com/springframeworkguru/springbootwebapp , that I have used in Docker homework. I downloaded it on my PC and uploaded it to this repo.
## Jenkinsfile - content with comments
~~~  
pipeline {
// Using jenkins slave with label "slave_homework"
	agent {
		node {
			label 'slave_homework'
	}
}
// Using Jenkins global tools - maven. Openjdk-11 is pre-installed on host, because this openjdk version couldn't be installed automaticly by Jenkins, but required by application.
	tools { 
        maven 'maven 3.6.3'
		
}
// Checkout openjdk and maven version
    stages { 
		stage('Checkout jdk and maven') {
			steps {
				sh "java -version"
				sh "mvn -v"
	}
}
// Compiling web-app from source code
		stage('Compiling web-app') {
			steps {
				sh "mvn package -f 'second_homework_java_web_app/springbootwebapp/pom.xml'"
	}
}
// Building image, where application will run for test
		stage('Building images') {
			steps {
				script {
					docker.build("image:build_${env.BUILD_ID}", "second_homework_java_web_app/")
        }
	}
}
// Running the container with web-application
		stage('Run container with web-app') {
			steps {
				sh "docker run -itd -p '8090:8080' --name 'container_${env.BUILD_ID}' 'image:build_${env.BUILD_ID}'"
	}
}
// Test application by "wget" and "curl", sending the request on localhost forwarded port
		stage('Test web-app') {
			steps {
				sh "wget -S localhost:8090"
				sh "curl localhost:8090"
		}
	}            
}
//Post actions that will be donne anyway, even after some stage fail - clearing workspace (deleting created container and images; clearing build directory)
	post('Cleaning workspace') { 
        always {
			sh "docker stop 'container_${env.BUILD_ID}' && docker rm 'container_${env.BUILD_ID}'"
			sh "docker rmi -f 'image:build_${env.BUILD_ID}'"
			cleanWs()
		}
	}
}
~~~
## Dockerfile - content with comments
~~~
# Using Ubuntu 18.04
FROM ubuntu:18.04

# Information about me
MAINTAINER denys naida <NaidaDV@nmu.one>

# Updating and upgrading; installing required soft
RUN apt-get update && apt-get upgrade -y && apt-get install openjdk-11-jdk -y

#Copying web-app artifact into image and starting web-app
COPY ./springbootwebapp/target/spring-boot-web-0.0.1-SNAPSHOT.jar /project/
ENTRYPOINT java -jar /project/spring-boot-web-0.0.1-SNAPSHOT.jar
~~~
## Workflow
Pipeline is successfull, application downloads from my git repo, maven builds it into jar localy, then jenkins build from dockerfile an image with this app, then from this image docker runs a container and finaly Jenkins test this app with "wget" and "curl". 
There are 5 scripted stages and 2 declarative (Checkot SCM and Post actions):

![](https://github.com/NaidaDV/jenkins_homework/blob/master/second_homework_java_web_app/screenshots/1.png)

Here are the results of the application tests:
![](https://github.com/NaidaDV/jenkins_homework/blob/master/second_homework_java_web_app/screenshots/2.png)

![](https://github.com/NaidaDV/jenkins_homework/blob/master/second_homework_java_web_app/screenshots/3.png)

Also homework folder include runtime_output.txt where you can find full successful output from Jenkins runtime.