   pipeline {
      agent any
      
      environment {
         DOCKER_IMAGE = "jusubyoo/total-app"
         DOCKER_TAG = "latest"
         EC2_HOST = "54.180.119.236"
         EC2_USER = "ubuntu"
         COMPOSE_FILE = "docker-compose.yml"
         CONTAINER_NAME = "total-app"
      }
      stages{
         stage('Checkout'){
            steps{
               echo 'Git Checkout'
               checkout scm
            }
         }
         
         stage('Gradlew Build') {
            steps{
               echo 'Gradle Build'
               sh """
                   chmod +x gradlew
                   ./gradlew clean build
                  """
            }
         }
         
         stage('Docker build'){
            steps {
               echo 'Docker Image build'
               sh """
                   docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                   
                   """
               
            }
         }
         
         stage('Docker Hub Login'){
            steps {
               echo 'Docker Hub Login'
               withCredentials([usernamePassword(
                  credentialsId: 'dockerhub_config',
                  usernameVariable: 'DOCKER_ID',
                  passwordVariable: 'DOCKER_PW'
               )]){
                  sh """
                      echo $DOCKER_PW | docker login -u $DOCKER_ID --password-stdin
                      docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                     """
               }
               
            }
            
            
         }
         /*
         stage('Deploy to EC2'){
            steps{
               sshagent(credentials:['SERVER_SSH_KEY']){
               sh """
                  ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                       docker stop total_app || true
                       docker rm total_app || true
                       docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                       docker run --name total_app -it -d -p 9090:9090 ${DOCKER_IMAGE}:${DOCKER_TAG}
                  """
                
               }
            }
            
         }
         */
         /*
         stage('Docker Compose Down'){
            steps{
               echo 'Docker-compose down'
               sh """
                  docker-compose -f ${COMPOSE_FILE} down || true
                  """
            }
         }
         
         
         stage('Docker Stop And RM'){
            steps{
               echo 'docker stop rm'
               sh """
                  docker stop ${CONTAINER_NAME} || true
                  docker rm ${CONTAINER_NAME} || true
                  docker pull ${DOCKER_IMAGE}
                  """
            }
            
         }   
         stage('Docker Compose up'){
            steps{
               echo 'Docker-compose up'
               sh """
                  docker-compose -f ${COMPOSE_FILE} up -d
                 """
            }
            
         }
         
         stage('Docker Run'){
            steps{
               echo 'Docker Run'
               sh """
                  docker stop ${CONTAINER_NAME} || true
                  docker rm ${CONTAINER_NAME} || true
                  
                  docker pull ${DOCKER_IMAGE}
                  
                  docker run --name ${CONTAINER_NAME} \
                  -it -d -p 9090:9090 \
                  ${DOCKER_IMAGE}
                  """
               
            }
         }
         */
         stage('Deploy docker-compose'){
            steps{
               sshagent(credentials:['SERVER_SSH_KEY']){
               sh """
                  ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
                       cd /home/ubuntu/app
                       docker-compose down
                       docker-compose pull
                       docker-compose up -d
EOF
                  """
                
               }
            }
         }
          
      
      }
      post {
            success{
               echo 'CI/CD 실행 성공'
            }
            failure {
               echo 'CI/CD 실행 실패'
               
            }
         }
   }