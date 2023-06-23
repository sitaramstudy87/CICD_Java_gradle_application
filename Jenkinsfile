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
		stage("pushing the helm charts to nexus"){
                        steps{
                                script{
					dir('kubernetes/') {
                                        	withCredentials([string(credentialsId: 'jenkins-nexus-dockerrepo', variable: 'docker_password')]) {
                                                	sh '''
								helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
								tar -czvf  myapp-${helmversion}.tgz myapp/
								curl -u admin:$docker_password http://192.168.10.7:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                                                	'''
						}
                                        }
                                }
                        }
                }
		stage('manual approval'){
			steps{
				script{
					timeout(10) {
						mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sitaramstudy87@gmail.com";
						input(id: "Deploy_Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    			}
				}
			}
		}
		stage('Deploying application on k8s cluster') {
			steps {
				script{
		                	//withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
					dir('kubernetes/') {
						sh 'helm upgrade --install --set image.repository="192.168.10.7:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
					}
					//}
				}
			}
		}
		stage('verifying app deployment'){
			steps{
				script{
					//withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
						sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'
					//}
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
