pipeline {
    agent { label 'Jenkins-Agent' }
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.0'
        DOCKER_USER = 'mirali94'
        DOCKER_PASS = credentials('dockerhub')  // Using Jenkins credentials for Docker Hub
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	   
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
       stage('Build & Push Docker Image') {
                steps {
                     script {
                        docker.withRegistry('',dockerhub){
                            docker_Image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        }
                       docker.withRegistry('',dockerhub){
                           docker_Image.push('${IMAGE_TAG}')
                           docker_Image.push('latest')
                       }
                    }      
                  }
            }
    }
}
