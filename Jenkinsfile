pipeline {
	agent any
	
	tools {
		maven 'Maven3.6.3'
	}

	environment {
		def serverIP = '3.85.194.215'
		def homeDir = '/home/sonar/tomcat8'
                def startServer = "${homeDir}/bin/startup.sh"
                def stopServer = "${homeDir}/bin/shutdown.sh"
	}

	stages {
		stage('Checkout') {
			steps {
				git url: 'https://github.com/SrilakshmiDoma14/petclinic.git'
			}
		}

		stage('Maven Build') {
			
			steps {
				sh label: '', script: 'mvn install'
			}
		}
		stage('Post Build Actions') {
			parallel {
				stage('Archive Artifacts') {
					steps {
						archiveArtifacts artifacts: 'target/*.?ar', followSymlinks: false
					}
				}

				stage('Test') {
					steps {
						junit 'target/surefire-reports/*.xml'
					}
				}
				
				stage('Publish Artifacts') {
					steps {
						nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/petclinic.war', type: 'war']], credentialsId: 'f5ae2c8a-9cae-4f1b-bd4a-5e79d67e456c', groupId: 'org.springframework.samples', nexusUrl: '3.87.136.76:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: '4.2.${BUILD_NUMBER}'"
					}
				}
				
				stage('Deploy') {
					steps {
                        			sh "scp -o StrictHostKeyChecking=no target/petclinic.war sonar@${serverIP}:/home/sonar/tomcat8/webapps/myweb.war"
                        			sh "ssh sonar@${serverIP} ${stopServer}"
                        			sh "ssh sonar@${serverIP} ${startServer}"
                    			}
				}
			}
		}
	}

	post {
		success {
			notify('Success')
		}
		failure {
			notify('Failed')
		}
		aborted {
			notify('Aborted')
		}
	}

}

def notify(status){
    emailext (
      to: "sravyamanjunath@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
