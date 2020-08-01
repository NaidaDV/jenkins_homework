## First homework - container with nginx inside
## Table of contents 
1. Dockerfile - content with comments
2. nginx.conf - changing nginx port
3. Jenkinsfile - content with comments
4. Workflow
## Dockerfile - content with comments 
~~~
# Using ubuntu 20.04
FROM ubuntu:20.04

# Information about me
MAINTAINER denys naida <NaidaDV@nmu.one>

# Updating, upgrading && installing reqired soft
RUN apt-get update && apt-get upgrade -y && apt-get install nginx -y && apt-get install net-tools -y && apt-get install openjdk-11-jdk -y && apt-get install sudo -y && apt-get install curl -y

# Adding user with privileges
RUN useradd -s /bin/bash -d /home/main_admin -m --no-log-init main_admin && usermod -aG sudo main_admin
RUN echo "main_admin:madmin" | chpasswd

# Adding user for nginx
RUN groupadd nginx_admin && useradd -g nginx_admin -s /bin/bash -d /etc/nginx -m  --no-log-init nginx_admin && usermod -aG www-data main_admin
RUN echo "nginx_admin:nadmin" | chpasswd

# Changing nginx.conf
RUN rm /etc/nginx/nginx.conf
COPY ./nginx.conf /etc/nginx/nginx.conf

#Adding rights to nginx_user
RUN chown -R nginx_admin /etc/nginx/ && chown -R nginx_admin /var/log/nginx/ && chown -R nginx_admin /var/lib/nginx/ && chown -R nginx_admin /run
RUN chmod -R u+w /etc/nginx/ && chmod -R u+w /var/log/nginx/ && chmod -R u+w /var/lib/nginx/ && chmod -R u+w /run

USER nginx_admin
CMD ["nginx", "-g", "daemon off;"]
~~~
## nginx.conf - changing nginx port
For changing default nginx port I have added this directive to http:
~~~
server {
        listen 8081;
        listen [::]:8081;

        server_name localhost 127.0.0.1;
        root /usr/share/nginx/html;
    }
~~~
So, now nginx working on port 8081, broadcasting it's default index.html
## Jenkinsfile - content with comments
~~~  
pipeline {
//Adding docker hub registry and credentials info; defining variable dockerImage
	environment { 
        	registry = "naidadv/courses_homework2" 
        	registryCredential = 'doc-hub-cred' 
        	dockerImage = '' 
}
//Using jenkins slave with label "slave_homework"
	agent {
		node {
        		label 'slave_homework'
	}
}
	stages {
//Building docker image from dockerfile in first_homework_container_with_nginx/
		stage('Building image') {
        		steps {
        			script {
                			dockerImage = docker.build("naidadv/courses_homework2:build_${env.BUILD_ID}", "first_homework_container_with_nginx/")
                }
	}
}
//Testing image by runnung container and sending request to forwarded port on localhost
		stage('Testing image') {
        		steps {
            			sh "docker run -itd -p '8081:8081' --name 'container_${env.BUILD_ID}' 'naidadv/courses_homework2:build_${env.BUILD_ID}'"
            			sh "curl localhost:8081"
            			sh "docker stop 'container_${env.BUILD_ID}' && docker rm 'container_${env.BUILD_ID}'"
        }
}       
//Pusing image to hub.docker.com 
		stage('Push image') {
        		steps {
        			script {
                			docker.withRegistry( 'https://registry.hub.docker.com/', registryCredential )  {
                    			dockerImage.push()
				}
			}
		}
	}    
}
//Post actions that will be donne anyway, even after some stage fail - clearing workspace (deleting created images and clearing build directory)
	post { 
        	always {
			sh "docker rmi $registry:build_${env.BUILD_ID}"
			sh "docker rmi -f 'registry.hub.docker.com/naidadv/courses_homework2:build_${env.BUILD_ID}'"
			cleanWs()
		}
	}
}
~~~
## Workflow
I have added my git repo and jenkinsfile path to job settings:

![](https://github.com/NaidaDV/jenkins_homework/blob/master/first_homework_container_with_nginx/screenshots/1.png)

There are 3 scripted stages and 2 declarative (Checkot SCM and Post actions):

![](https://github.com/NaidaDV/jenkins_homework/blob/master/first_homework_container_with_nginx/screenshots/2.png)

"Nginx is running inside container" proofs:

![](https://github.com/NaidaDV/jenkins_homework/blob/master/first_homework_container_with_nginx/screenshots/3.png)

Also homework folder include runtime_output.txt where you can find full successful output from Jenkins runtime.
Dockerhub is: https://hub.docker.com/repository/docker/naidadv/courses_homework2
