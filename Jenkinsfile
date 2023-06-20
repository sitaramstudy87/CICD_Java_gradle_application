pipeline{
	agent any
	stages{
		stage("Sonar Quality check"){
		//	agent{
		//		docker{
	//				image 'openjdk:11'
	//			}
	//		}
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
	}
}
