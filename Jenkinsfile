pipeline {

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: spring-boot-app
    image: juanllorenzogomis/jenkins-nodo-java-bootcamp:3.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'spring-boot-app'
        }
    }


	environment {
		DOCKERHUB_CREDENTIALS=credentials("jenkins_dockerhub")
		DOCKER_IMAGE_NAME="juanllorenzogomis/spring-boot-app"
	}

	stages {
		stage("Prepare environment"){
			steps {
				echo "version de java"
				sh 'java -version'
				echo "version de maven"
				sh 'mvn -version'
			}
		}

		stages {
		stage("Compile"){
			steps {
				sh "mvn clean package -DskipTests"
			}
		}


	}

	post {
		always {
			sh "docker logout"
		}
	}

}
