pipeline{
	agent any
	environment {
        VERSION = "${env.BUILD_ID}"
        }
	stages{
		stage("Sonar quality check"){
		//agent {
                //docker {
                //    image 'openjdk:11'
                //}
            	//}
			steps{
				script{
					withSonarQubeEnv(credentialsId: 'sonar-jenkin-token') {
					sh 'chmod +x gradlew'
					sh './gradlew sonarqube' 
					}
					timeout(time: 1, unit: 'HOURS') {
 						def qg = waitForQualityGate()
						if (qg.status != 'OK') {
							error "Pipeline aborted due to qualitygate failer : ${qg.status}"
						}
					}
				}	
			}
		}	
		stage("Docker Build and push to repo") {
                        steps{
                                script{
                                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                                                sh '''
                                                        docker build -t 192.168.122.171:8083/springapp:${VERSION} .
                                                        docker login -u admin -p $docker_password 192.168.122.171:8083
                                                        docker push 192.168.122.171:8083/springapp:${VERSION}
                                                        docker rmi 192.168.122.171:8083/springapp:${VERSION}
                                                '''
                                        }
                                }
                        }
                }
		stage("Identify the misconfig using datree in helm charts") {
			steps{
				script{
					dir('kubernetes/') {
						withEnv(['DATREE_TOKEN=473b3594-4ccb-4e57-ac47-94481af7789f']) {
    							sh 'helm datree test myapp/'
						}
					} 						
				}
			}
		}

		stage("Pushing Helm to nexus helm-hosted repo") {
                        steps{
                                script{
                                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                                              dir('kubernetes/') {
						sh  '''
							helmversion=$(helm show chart myapp |grep versio|awk '{print $2}')
							tar -czvf myapp-${helmversion}.tgz myapp/
							 curl -u admin:$docker_password http://192.168.122.171:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
						'''
						}
                                        }
                                }
                        }
                }
		post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sitaramstudy87@gmail.com";  
		 }
	   }
	}
	
}
