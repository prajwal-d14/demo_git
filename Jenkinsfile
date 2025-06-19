
pipeline {
    agent none
    parameters {
        choice(name: 'Environment', choices: ['qa', 'prod'], description: 'Select environment to deploy')
    }
  
    stages {
        stage('SCM Checkout') {
            agent { label 'compile' }
            steps {
                git branch: 'main', url: 'https://github.com/prajwal-d14/java-project.git'
            }
        }

        stage('Build') {
            agent { label 'compile' }
            steps {
                sh '''
                    mvn clean install
                    sleep 5
                    cp /home/ubuntu/workspace/java-project/target/demo-0.0.1-SNAPSHOT.war /home/ubuntu/builds/
                '''
            }
        }

        stage('Deploy') {
            agent { label "${params.Environment.toLowerCase()}" }
            steps {
                sh '''
                   scp ubuntu@172.31.7.95:/home/ubuntu/builds/demo-0.0.1-SNAPSHOT.war ~ && sudo mv ~/demo-0.0.1-SNAPSHOT.war /opt/tomcat/webapps/
                   sleep 5
                   sudo systemctl restart tomcat.service
                   sudo systemctl status tomcat.service
                   
                '''
            }
        }

        stage('Test') {
			agent { label "${params.Environment.toLowerCase()}" }
			steps {
				script {
					def ip = sh(script: "curl -s http://checkip.amazonaws.com", returnStdout: true).trim()
					def appUrl = "http://${ip}:8080/demo-0.0.1-SNAPSHOT"
            
					echo "Test was successful"
					echo "Access the deployed application from the link: ${appUrl}"

					emailext (
					subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
					body: """<p>Job '${env.JOB_NAME} [#${env.BUILD_NUMBER}]' has finished with status: <b>${currentBuild.currentResult}</b></p>
                         <p>Application is deployed and accessible at: <a href="${appUrl}">${appUrl}</a></p>
                         <p>See the Jenkins console output: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
					mimeType: 'text/html',
					to: 'prajwaldoddananjaiah@gmail.com'
					)
				}
			}
		}

    }
}
