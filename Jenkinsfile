pipeline {
	agent none

		stages {
			stage('SCM Checkout') {
				agent { label 'compile' }
				steps {
					git branch: 'main', url: 'https://github.com/prajwal-d14/demo_git.git'
				}
			}

			stage('Build') {
				agent { label 'compile' }
				steps {
					sh '''
						mvn clean install
						sleep 20
						cp /home/ubuntu/workspace/Build/target/demo-0.0.1-SNAPSHOT.war /home/ubuntu/builds/
						'''
				}
			}

			stage('Deploy') {
				agent { label 'deploy' }
				steps {
					script {
						def serverIp = ''
							if (params.Environment == 'QA') {
								serverIp = '172.31.7.95' // QA server IP
							} else if (params.Environment == 'PROD') {
								serverIp = '172.31.10.88' // Replace with actual PROD server IP
							} else {
								error("Unknown environment: ${params.Environment}")
							}

						echo "Deploying to ${params.Environment} server at IP: ${serverIp}"

							sh """
							scp ubuntu@${serverIp}:/home/ubuntu/builds/demo-0.0.1-SNAPSHOT.war ~
							sudo mv ~/demo-0.0.1-SNAPSHOT.war /opt/tomcat/webapps/
							sleep 5
							sudo systemctl restart tomcat.service
							sudo systemctl status tomcat.service
							"""
					}
				}
			}


			stage('Test') {
				agent { label 'deploy' }
				steps {
					sh '''
						echo test was successful
						ip=$(curl -s http://checkip.amazonaws.com)
						echo "Access the Deployed application from the link http://$ip:8080/demo-0.0.1-SNAPSHOT"
						'''
				}
			}
		}

	post {
		success {
			script {
				def appUrl = "http://$ip:8080/demo-0.0.1-SNAPSHOT"

					emailext(
							subject: "Deployment Successful",
							body: """
							Hello Team,

							The deployment of the application was successful.

							You can access the app here:
							${appUrl}

							Build URL: ${env.BUILD_URL}
							""",
							to: 'prajwaldoddananjaiah@gmail.com'
						)
			}
		}

		failure {
			emailext(
					subject: "Deployment Failed",
					body: """
					Hello Team,

					The Jenkins job has failed.

					Please check the console logs here:
					${env.BUILD_URL}
					""",
					to: 'prajwaldoddananjaiah@gmail.com'
				)
		}
	}
}

