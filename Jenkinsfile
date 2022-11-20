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
					dir('./kubernetes/') {
    						sh 'helm datree test myapp/'
					} 						
				}
			}
		}
	}
	
}
