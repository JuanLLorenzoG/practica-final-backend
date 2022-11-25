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
    command:
    - cat
    imagePullPolicy: IfNotPresent
    tty: true
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
		DOCKER_HUB="jenkins_dockerhub"
		DOCKER_HUB_USER="juanllorenzogomis"
		DOCKER_HUB_PASS="Dockerhub43v3r"
		DOCKERHUB_CREDENTIALS=credentials("jenkins_dockerhub")
		DOCKER_IMAGE_NAME="juanllorenzogomis/prueba-final-backend"
		VERSION=$(readMavenPom().getVersion())
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
				sh 'mvn package -DskipTests'
			}
		}

		stage("Build & Push"){
			steps { 
				container('kaniko'){
					echo "Aqui se construye la imagen"
					script {
						withCredentials([usernamePassword(credentialsId: "jenkins_dockerhub", passwordVariable: "juanllorenzogomis", usernameVariable: "Dockerhub43v3r")]) {
							AUTH = sh(script: """echo -n "${DOCKER_HUB_USER}:${DOCKER_HUB_PASS}" | base64""", returnStdout: true).trim()
							command = """echo '{"auths": {"https://index.docker.io/v1/": {"auth": "${AUTH}"}}}' >> /kaniko/.docker/config.json"""
							sh("""
							set +x
							${command}
							set -x
							""")
							sh "/kaniko/executor --context `pwd` --destination ${DOCKER_IMAGE_NAME}:${VERSION} --cleanup"
						}
					}
				}
			}
		}

	}

	post {
		always {
			sh "docker logout"
		}
	}

}
