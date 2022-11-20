pipeline{
	agent any
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
                                                        docker build -t 192.168.122.171:8081/springapp:${versin} .
                                                        docker login -u admin -p $docker_password 192.168.122.171:8081
                                                        docker push 192.168.122.171:8081/springapp:${versin}
                                                        docker rmi 192.168.122.171:8081/springapp:${versin}
                                                '''
                                        }
                                }
                        }
                }

	}
	
}
