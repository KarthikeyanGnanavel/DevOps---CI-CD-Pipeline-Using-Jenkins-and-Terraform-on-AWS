pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        POSTMAN_ACCESS_KEY = credentials('POSTMAN_ACCESS_KEY')
    }
    stages {
	    stage('Build Start Email Notification'){
            steps{
                emailext (
				        	subject: "Jenkins PipeLine: ${JOB_NAME} Started - Build Number: #${BUILD_NUMBER}", 
					        body: '''<html>
						        		<body>
							        		<p><b>Pipeline Name:</b> ${JOB_NAME}</p>
								        	<p><b>Build Number:</b> ${BUILD_NUMBER}</p>
									        <p>Please check the <a href="${BUILD_URL}">console output</a> here.</p>
								        </body> 
							        </html>''',
				    	    to: 'karthikgnanavel88@gmail.com',
					        from: 'jenkins@gmail.com',
					        replyTo: 'jenkins@gmail.com', 
					        mimeType: 'text/html'
				        )
            }
        }    
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/KarthikeyanGnanavel/DevOps---CI-CD-Pipeline-Using-Jenkins-and-Terraform-on-AWS.git'
            }
        }
        // stage("Sonarqube Analysis") {
        //     steps {
        //         withSonarQubeEnv('sonar-server') {
        //             sh 'docker start sonar || docker run -d --name sonar -p 9000:9000 sonarqube:latest'
        //             sh '''
        //             $SCANNER_HOME/bin/sonar-scanner \
        //             -Dsonar.projectName=Amazon \
        //             -Dsonar.projectKey=Amazon
        //             '''
        //         }
        //     }
        // }
        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: true, credentialsId: 'jenkins'
        //         }
        //     }
        // }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        // stage('OWASP FS SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        // stage('TRIVY FS SCAN') {
        //     steps {
        //         sh "trivy fs . > trivyfs.txt"
        //         archiveArtifacts artifacts: 'trivyfs.txt'
        //     }
        // }
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh """
                        docker build -t karthik8898/amazon-clone:latest .
                        docker push karthik8898/amazon-clone:latest
                        """
                    }
                }
            }
        }
        // stage("TRIVY"){
        //     steps{
        //         sh "trivy image karthik8898/amazon-clone:latest > trivyimage.txt" 
        //     }
        // }
        stage('Deploy to container'){
            steps{
				sh 'docker stop amazon-clone || true' 
				sh 'docker rm amazon-clone || true' 
                sh 'docker run -d --name amazon-clone -p 3000:3000 karthik8898/amazon-clone:latest'
                sh 'docker ps'
            }
		}
		stage('Postman API Testing') {
            steps {
                script {
                    sh 'npm install -g newman-reporter-html'
                    // Run Postman collection and capture output in text and report in JUnit/HTML
                    sh 'newman run https://api.postman.com/collections/33996834-e6157687-8644-4f85-aeee-fd6796eaaebc?access_key=$POSTMAN_ACCESS_KEY --reporters junit,html --reporter-junit-export newman-report.xml --reporter-html-export newman-report.html > newman-output.txt'
                }
                // Archive the output file and the generated reports
                archiveArtifacts artifacts: 'newman-output.txt, newman-report.xml, newman-report.html'
                
                // Optionally, publish HTML report in Jenkins
                publishHTML(target: [
                    reportName: 'Postman Test Report',
                    reportDir: '.',
                    reportFiles: 'newman-report.html'
                ])
            }
        }
   }
   post {
		        always {
			        	emailext (
				        	subject: "Jenkins PipeLine: ${JOB_NAME} Completed - Build Number: #${BUILD_NUMBER}", 
					        body: '''<html>
						        		<body>
						    	    	    <p><b>Pipeline Name:</b> ${JOB_NAME}</p>
							            	<p><b>Build Status:<a style=color:blue>${BUILD_STATUS}!!</a></b></p>
								    	    <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
									        <p>Please check the <a href="${BUILD_URL}">console output</a> to view the results.</p>
								        </body>  
							         </html>''',
				    	    to: 'karthikgnanavel88@gmail.com',
					        from: 'jenkins@gmail.com',
					        replyTo: 'jenkins@gmail.com', 
					        mimeType: 'text/html'
				        )
		               }
	    }
}

