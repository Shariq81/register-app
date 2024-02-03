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
          JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
    stage("Trivy Scan"){
      steps {
        script {
                sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy shariq81/register-app-pipeline:latest -q --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
          }
        }
      }
    }
    stage("Cleanup Artifacts"){
      steps {
        script {
          sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
          sh "docker rmi ${IMAGE_NAME}:latest"
         }
       }
    }
    stage("Trigger CD Pipeline"){
      steps {
        script {
          sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'Content-Type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-54-189-189-57.us-west-2.compute.amazonaws.com:8080/job/register-app-pipeline/buildWithParameters?token=gitops-token'"
        }
      }
    }
  }
}