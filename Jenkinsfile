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
        DOCKER_PASS = 'dockerhub'  // Using Jenkins credentials for Docker Hub
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
                    // Build Docker image
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")

                    // Push Docker image with tags
                    docker.withRegistry('', DOCKER_PASS) {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mirali94/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
         stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
        
    }
}
