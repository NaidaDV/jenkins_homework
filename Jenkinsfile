pipeline {
    environment { 
        registry = "naidadv/courses_homework2" 
        registryCredential = '74b9705b-baac-41a9-a3b9-e495f96c666e' 
        dockerImage = '' 
    }
    agent any 
            stages {
                
    stage('Building image') {
        steps {
            script {
                dockerImage = docker.build("naidadv/courses_homework2:build_${env.BUILD_ID}", "./nginx")
                }
	}
}
                    
    stage('Testing image') {
        steps {
            sh "docker run -itd -p '8081:8081' --name 'container_${env.BUILD_ID}' 'naidadv/courses_homework2:build_${env.BUILD_ID}'"
            sh "curl localhost:8081"
            sh "docker stop 'container_${env.BUILD_ID}' && docker rm 'container_${env.BUILD_ID}'"
        }
}
                            
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
	post { 
        always {
		sh "docker rmi $registry:build_${env.BUILD_ID}"
		cleanWs()
		}
	}
}
