pipeline {
  agent any
  tools {
    jdk 'jdk17'
    nodejs 'node16'
  }
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
    APP_NAME = "dip-reddit-app"
    RELEASE = "1.0.0"
    IMAGE_NAME = "myacr1001.azurecr.io"+"/"+"${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
  }
  stages {
    stage('clear workspace') {
      steps {
       cleanWs() 
      }      
    }
    stage('checkout from git') {
      steps {
       sh 'git clone https://github.com/dipanjanr640/a-reddit-clone.git' 
      }      
    }
    stage('sonarqube analysis') {
      steps {
       //sonar analysis - using sonar-server (management system) it run command to analysis by sonarqube with sonar tool(management tool)
        withSonarQubeEnv('sonar-server') {
         sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=eddit-clone-ci \
         -Dsonar.projectKey=eddit-clone-ci'''
                 }
      }      
    }
    stage('quality gate') {
      steps {
        //quality gate checks using sonar-token
        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
      }
    stage('dependecies install') {
      steps {
        //as it is node js project we have to run npm install command
        sh "npm install"
        }
      }
    stage('trivy code scan') {
      steps {
        //trivy already installed on local system where jenkins agent is running. trivy scane is need to checking vulnerability
        //The trivy fs command is used to scan the filesystem of a container image for vulnerabilities.
        sh "trivy fs . > trivyfs.txt"
        }
      }
    stage('build docker image') {
      steps {
        sh '''
        cd a-reddit-clone
        docker build -t $IMAGE_NAME:latest .'''
        }
      }
    stage('push to az acr') {
      steps {
        azureCLI commands: [[exportVariablesString: '', script: 'az acr login --name myacr1001']], principalCredentialId: 'AzureSPNew'
        sh '''
        docker push $IMAGE_NAME:latest
        '''
        }
      }
    stage('trivy image scan') {
      steps {
        sh "trivy image $IMAGE_NAME:latest --scanners vuln --exit-code 0 > trivyimage.txt"
        //Or you can use docker trivy image to scan your image
        //sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/reddit-clone-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
        }
      }
    stage('clear build image') {
      steps {
        sh "docker rmi $IMAGE_NAME:latest"
        }
      }
    }
  post {
    always {
      emailext attachLog: true,
               subject: "'${currentBuild.result}'",
               body: "Project: ${env.JOB_NAME}<br/>" +
                   "Build Number: ${env.BUILD_NUMBER}<br/>" +
                   "URL: ${env.BUILD_URL}<br/>",
               to: 'dipanjanr640@gmail.com',                              
               attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    }
  }
  }

