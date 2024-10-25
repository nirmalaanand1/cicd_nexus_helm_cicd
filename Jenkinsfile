pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage('sonar quality check'){
            agent{
                docker{
                    image 'maven'
                    args '-u root'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh 'mvn clean package sonar:sonar'
                   }
                }
            }
        }
        stage('Quality Gate Status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }

        }
        stage('Docker build and Docker push to nexus repo'){
            steps{
                script{
                  withCredentials([string(credentialsId: 'nexus_pwd', variable: 'nexus_creds')]) {
                  sh '''
                    docker build -t 13.53.194.11:8083/springapp:${VERSION} .

                    docker login -u admin -p $nexus_creds 13.53.194.11:8083

                    docker push 13.53.194.11:8083/springapp:${VERSION} 

                    docker rmi 13.53.194.11:8083/springapp:${VERSION} 
                  '''
                  }
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "missdevi989@gmail.com";  
		}
	}
}