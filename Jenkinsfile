pipeline {
    agent any
    environment {
        TEST_VAR="CHANGE_ME"

    }
   
    stages {

    // Tests
    stage('Unit Tests') {
      steps{
        script {
          sh 'echo "In Unit Tests"'
        }
      }
    }
        
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          sh 'echo "In Building image"'
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
        script {
          sh 'echo "Pushing to ECR"'
        }
        }
      }
      
    stage('Deploy') {
     steps{
        script {
          sh 'echo "Deploy"'
        }        }
      }      
      
    }
}