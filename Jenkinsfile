pipeline {
  agent { label 'Jenkins-Agent' }
  tools { 
    jdk 'Java17'
    maven 'Maven3'
  }
  environment {
          App_Name = 'register-app-pipeline'
          Release_Version = '1.0.0'
          DOCKER_USER = 'shariq81'
          DOCKER_PASS = 'dockerhub'
          IMAGE_NAME = "${DOCKER_USER}" + "/" + "${App_Name}"
          IMAGE_TAG = "${Release_Version}-${BUILD_NUMBER}"
  }
  stages{
    stage("Cleanup Workspace"){
      steps {
        deleteDir()
      }
    }
    stage("Checkout from SCM"){
      steps {
        git branch: 'main', credentialsId: 'github', url: 'https://github.com/Shariq81/register-app'
      }
    }
    stage("Build Application"){
      steps {
        sh "mvn clean package"
      }
    } 
    stage("Test Application"){
      steps { 
        sh "mvn test"
      }
    }
    stage("SonarQube Analysis"){
      steps {
        script {
          withSonarQubeEnv(credentialsId: 'jenkins-sonaqube-token') {
          sh "mvn sonar:sonar"
        }
      }
    }
  }
  stage("Quality Gate"){
    steps {
      script {
             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonaqube-token'
        }
      }
    }
    stage("Build & Push Docker Image"){
      steps {
        script {
          docker.withRegistry('',DOCKER_PASS) {
            docker_image = docker.build "${IMAGE_NAME}"
      }
      docker.withRegistry('',DOCKER_PASS) {
        docker_image.push "${IMAGE_TAG}"
        docker_image.push "latest"
          }
        }
      }
    }
  }
}
