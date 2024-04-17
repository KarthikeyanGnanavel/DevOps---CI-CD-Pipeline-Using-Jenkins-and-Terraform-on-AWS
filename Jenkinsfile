pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
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
				    	    to: 'karthikgnanavel88@gmail.com,buvimasi98@gmail.com',
					        from: 'jenkins@gmail.com',
					        replyTo: 'jenkins@gmail.com', 
					        mimeType: 'text/html'
				        )
            }
        }
        stage('Cleaning Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Github', url: 'https://github.com/KarthikeyanGnanavel/DevOps---CI-CD-Pipeline-Using-Jenkins-and-Terraform-on-AWS.git']])
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
					sh 'docker start sonar'
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                    -Dsonar.projectKey=Amazon '''
                }
            }
        }
         stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t amazon-clone ."
                       sh "docker tag amazon-clone karthik8898/amazon-clone:latest "
                       sh "docker push karthik8898/amazon-clone:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image karthik8898/amazon-clone:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
				sh 'docker stop amazon-clone'
				sh 'docker rm amazon-clone'
                sh 'docker run -d --name amazon-clone -p 3001:3000 karthik8898/amazon-clone:latest'
            }
        }
        stage('Postman API Testing') {
            steps {
                sh 'newman run https://api.postman.com/collections/33996834-914fc6e6-f403-444b-8687-7ae2943591ba?access_key=PMAT-01HVPREVMNMH4H917P2Q2B5SN3'
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
				    	    to: 'karthikgnanavel88@gmail.com,shrikanaga5@gmail.com',
					        from: 'jenkins@gmail.com',
					        replyTo: 'jenkins@gmail.com', 
					        mimeType: 'text/html'
				        )
		               }
	    }
}
