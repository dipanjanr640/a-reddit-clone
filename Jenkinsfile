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
    // stage('sonarqube analysis') {
    //   steps {
    //    //sonar analysis - using sonar-server (management system) it run command to analysis by sonarqube with sonar tool(management tool)
    //     withSonarQubeEnv('sonar-server') {
    //      sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=eddit-clone-ci \
    //      -Dsonar.projectKey=eddit-clone-ci'''
    //              }
    //   }      
    // }
    // stage('quality gate') {
    //   steps {
    //     //quality gate checks using sonar-token
    //     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
    //     }
    //   }
    stage('dependecies install') {
      steps {
        //as it is node js project we have to run npm install command
        sh "npm install"
        }
      }
    stage('trivy scan') {
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
        // sh '''
        // az acr login --name myacr1001 --username 811ca00c-bd20-4f28-9949-ad91263a5371 --password FWt8Q~QBT4exjwUGj5PW22KWy-4UpjazUo.oiaZu
        // docker push docker/dip-reddit-app:latest'''
        azureCLI commands: [[exportVariablesString: '', script: 'az acr login --name myacr1001'], [exportVariablesString: '', script: 'docker push myacr1001.azurecr.io/dip-reddit-app:latest']], principalCredentialId: 'AzureSPNew'
        }
      }
    }
  }

