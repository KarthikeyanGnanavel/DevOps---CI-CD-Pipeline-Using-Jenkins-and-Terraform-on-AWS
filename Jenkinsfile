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
        stage('Cleaning Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/KarthikeyanGnanavel/DevOps---CI-CD-Pipeline-Using-Jenkins-and-Terraform-on-AWS.git'
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
    }
}
