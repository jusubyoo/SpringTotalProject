pipeline {
	agent any
	
	environment {
		SERVIER_IP = "54.180.119.236"
		SERVER_USER = "user"
		APP_DIR = "~/app"
		JAR_NAME = "SpringTotalProject-0.0.1-SNAPSHOT.war"
	}
	
	stages {	
		/*stage('Check Git Info') {
			steps {
				sh '''
				   echo "===Git Info==="
				   git branch
				   git log -1
				   '''
			}
		}*/
		
		stage('Check Out') {
			steps {
				echo 'Git Checkout'
				checkout scm
			}
		}
		
		// gradle build => war 파일을 다시 생성
		stage('Gradle Permission') {
			steps {
				sh '''
				   chmod +x gradlew
				   '''
			}
		}
		
		// build 시작
		stage('Gradle Build') {
			steps {
				sh '''
				   ./gradlew clean build
				   '''
			}
		}
		
		// war 파일 전송 = rsync / scp 1
		stage('Deploy = rsync') {
			steps {
				sshagent(credentials:['SERVER_SSH_KEY']) {
					sh '''
					   rsync -avz -e 'ssh -o StrictHostKeyChecking=no' build/libs/*.war ${SERVER_USER}@${SERVER_IP}:${APP_DIR} 
					   '''
					   
				}
			}
		}
		
		// 실행 명령
		stage('Run Application') {
			steps {
				sshagent(credentials:['SERVER_SSH_KEY']) {
					sh '''
					   ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_IP} << 'EOF'
					   	 pkill -f 'java -jar' || true
					   	 nohup java -jar ${APP_DIR}/${JAR_NAME} > log.txt 2>&1 &
					   EOF
					   '''
					   
				}
			}
		}
	}
}