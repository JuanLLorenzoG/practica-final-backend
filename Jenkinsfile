pipeline {

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: spring-boot-app
    image: juanllorenzogomis/jenkins-nodo-java-js-bootcamp:1.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    volumeMounts:
    - mountPath: /kaniko/.docker
      name: kaniko-docker-cfg
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
  - name: kaniko-docker-cfg
    hostPath:
      path: /kaniko/.docker
    commad:
    - sleep
    args:
    - 99d
    projected:
      sources:
      - secret:
        name: regcred
        items: 
        - key: .dockerconfigjson
          path: config.json
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

		stage("Compile"){
			steps {
				sh "mvn clean compile -DskipTests"
			}
		}

		stage("Unit Tests"){
			steps{
				sh "mvn test"
				junit "target/surefire-reports/*.xml"
			}
		}
		
		stage("JaCoCo Test"){
			steps{
				//sh "mvn test"
				jacoco()
			}
		}

		stage("Quality Tests") {
			steps {
			        script {
					withSonarQubeEnv("sonarqube-server"){
						sh 'set +x mvn sonar:sonar \
						-Dsonar.projectKey=Practica-Final-Backend \
						-Dsonar.host.url=http://localhost:9000 \
						-Dsonar.login=sqp_c9a1f2d6848a11ec647e8c29e06423742b131d69 set -x'
					}
				}
			}
		}

		stage("Package"){
			steps {
				sh 'mvn package'
			}
		}

	}

	post {
		always {
			sh "docker logout"
		}
	}

}
