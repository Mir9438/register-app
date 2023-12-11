pipeline {
    agent { label 'Jenkins-Agent' }
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'Github', url: 'https://github.com/Mir9438/register-app.git'
            }
        }
        
        stage('Build Application') {
            steps {
                  sh 'mvn clean package'
            }
        }
       stage('Test Application') {
            steps {
                  sh 'mvn test'
            }
        }
       stage('SonarQube Analysis') {
            steps {
                  withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar'
                    }      
                  }
            }
            stage('Quality Gate') {
                steps {
                     script {
                        waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                    }      
                  }
            }
     
    }
}
