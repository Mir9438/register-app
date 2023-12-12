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
	JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
	   
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
         stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-127-136-75.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }

    post {
       failure {
             emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                      subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                      mimeType: 'text/html',to: "mir.ali19912@gmail.com"
      }
      success {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                     mimeType: 'text/html',to: "mir.ali19912@gmail.com"
            }     
        }    
    }
}
	
