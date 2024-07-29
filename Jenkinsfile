pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = '/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar-scanner/'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/pradnyeo/Amazon-app-Deployment-using-terraform-and-jenkins.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                    -Dsonar.projectKey=Amazon '''
                }
            }
        }
         stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar_token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Docker Build & tag"){
            steps{
                sh 'docker build -t amazon-clone .'
                sh "docker tag amazon-clone pradnyeo/amazon-clone:latest "
            }
        }
        stage("Docker_Push_Image") {
            steps {
                withCredentials([string(credentialsId: 'Dockerhub_Cred', variable: 'DockerHub')]) {
                    sh 'docker login -u pradnyeo -p ${DockerHub}'
                }
                sh 'docker push pradnyeo/amazon-clone:latest'
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image pradnyeo/amazon-clone:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon-clone -p 3000:3000 pradnyeo/amazon-clone:latest'
            }
        }
    }
}
