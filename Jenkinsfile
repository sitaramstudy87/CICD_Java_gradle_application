pipeline{
	agent any
	 environment{
		VERSION = "${env.BUILD_ID}"
		}
	stages{
		stage("Sonar Quality check"){
			 //agent {
                	//	docker {
                    	//		image 'openjdk:11'
                	//	}
            		//}
			steps{

				script{
					withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
 						sh 'chmod +x gradlew'
						sh './gradlew sonarqube'
					
					}		
					
					timeout(time: 1, unit: 'HOURS') {
                      				def qg = waitForQualityGate()
                      				if (qg.status != 'OK') {
                          			 	error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      				}
                    			}
			
				}
			}
		}
		stage("docker build & docker push"){
			steps{
				script{
					withCredentials([string(credentialsId: 'jenkins-nexus-dockerrepo', variable: 'docker_password')]) {
						sh '''
							docker build -t 192.168.10.7:8083/springapp:${VERSION} .
							docker login -u admin -p $docker_password 192.168.10.7:8083 
 							docker push  192.168.10.7:8083/springapp:${VERSION}
							docker rmi 192.168.10.7:8083/springapp:${VERSION}
						'''
					}
				}
			}
		}
		stage('indentifying misconfigs using datree in helm charts'){
			steps{
				script{
					dir('kubernetes/') {
						withEnv(['DATREE_TOKEN=ac663fd6-3f45-4c42-b488-9a6391ad8a4e']) {
							sh 'helm datree test myapp/'
						}
					}
				}
			}
		}	
	}
	post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8',  mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sitaramstudy87@gmail.com";  
	 	}
	}
	
}
