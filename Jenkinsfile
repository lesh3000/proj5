def value(){
    return sh(script: "docker ps -aq", returnStdout: true).trim()
}

def all(){
    return sh(script: "docker images -q", returnStdout: true).trim()
}

pipeline {
  environment {
    registry = "dmitrylesh/sklearn"
    registryCredential = 'dockerhub'
    dockerImage = ''
    USER_CREDENTIALS = credentials('aws')
    
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        checkout scm
      }
    }
    stage('Remove all') {
        steps{

        sh "docker rm -vf ${value()}"
        sh "docker rmi -f ${all()}"
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps{
         script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
     stage('pull image and train model') {
      steps{
        sh "docker pull $registry:$BUILD_NUMBER"
        sh "docker run -e secret_key=$USER_CREDENTIALS_PSW -e key_id=$USER_CREDENTIALS_USR --rm $registry:$BUILD_NUMBER"
      }
    }
  }
}

