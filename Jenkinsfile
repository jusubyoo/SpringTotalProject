pipeline {
    agent any

    environment {
	   DOCKER_IMAGE = "jusubyoo/total-app"
	   DOCKER_TAG = "latest"
	   EC2_HOST = "54.180.119.236"
	   EC2_USER = "ubuntu"	
	}
	
    stages {
		// GIT 연결 => 주소 1
        stage('Checkout') {
            steps {
                echo 'Git Checkout'
                checkout scm
            }
        }
        // 배포판 만들기 
        stage('Gradlew Build') {
			steps {
				echo 'Gradle Build'
				sh '''
				    chmod +x gradlew
				    ./gradlew clean build -x test
				   '''
			}
		}
		
		stage('Docker Build') {
			steps {
				echo 'Docker Image Build'
				sh '''
				    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
				   '''
			}
		}
		
		stage('Docker Hub Login') {
			steps {
				echo 'DockerHub Login'
				withCredentials([usernamePassword(
					credentialsId: 'dockerhub-config',
					usernameVariable: 'DOCKER_ID',
					passwordVariable: 'DOCKER_PW'
				)]){
					sh '''
					   echo "DOCKER_ID=$DOCKER_ID,DOCKER_PW=$DOCKER_PW"
					   echo "$DOCKER_PW" | docker login -u "$DOCKER_ID" --password-stdin
					   docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
					   '''
				}
			}
		}
		
		stage('Deploy to EC2') {
			steps {
			  // Manage => SSH Agent 설치 = jenkins 다시 실행 
			  sshagent(credentials: ['SERVER_SSH_KEY']) {
				sh """
				   ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
				       docker stop total-app || true
				       docker rm total-app || true
				       docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
				       docker run --name total-app -it -d -p 9090:9090 ${DOCKER_IMAGE}:${DOCKER_TAG}
				   EOF
				   """
			  }
			}
		}
		
		/*stage('Docker Compose Down') {
			steps {
				echo 'docker-compose down'
				sh '''
				     docker-compose -f ${COMPOSE_FILE} down || true
				   '''
			}
		}*/
		
		/*
		stage('Docker Stop And RM'){
			steps {
				echo 'docker stop rm'
				sh '''
				    docker stop ${CONTAINER_NAME} || true
				    docker rm ${CONTAINER_NAME} || true
				    docker pull ${IMAGE_NAME}
				   '''
			}
		}
		
		
		stage('Docker Compose UP') {
			steps {
				echo 'docker-compose up'
				sh '''
				    docker-compose -f ${COMPOSE_FILE} up -d
				   '''
			}
		}
		*/
		/*stage('Docker Run') {
			steps {
				echo 'Docker Run'
				sh '''
				    docker stop ${CONTAINER_NAME} || true
				    docker rm ${CONTAINER_NAME} || true
				    
				    docker pull ${IMAGE_NAME}
				    
				    docker run --name ${CONTAINER_NAME} \
				    -it -d -p 9090:9090 \
				    ${IMAGE_NAME}
				   '''
			}
		}*/
    }
    
    post {
		success {
			echo 'CI/CD 실행 성공'
		}
		failure {
			echo 'CI/CD 실행 실패'
		}
	}
}
