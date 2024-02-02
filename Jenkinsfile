pipeline {
  agent { label 'Jenkins-Agent' }
  tools { 
    jdk 'Java17'
    maven 'Maven3'
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
    } // Missing closing brace added here
    stage("Test Application"){
      steps { 
        sh "mvn test"
      }
    }
  }
}
