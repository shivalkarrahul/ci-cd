pipeline {
    agent any

    environment {
        // Initialize ENVIRONMENT with a default value, but allow it to be overridden by the parameter
        ENVIRONMENT = "${ENVIRONMENT}"
        NODE_IMAGE = "circleci/node:16.13.1-bullseye-browsers" 
    }

   
    stages {

    stage('Initialization') {
      steps{
        script {
          sh 'echo "Environment =  $ENVIRONMENT"'
        }
      }
    }

    stage('Unit Tests') {
      steps{
        script {
          sh 'echo "In Unit Tests $ENVIRONMENT"'
          sh """
          docker run --rm --user root -v "$WORKSPACE":/home/circleci/app $NODE_IMAGE /bin/bash -c "cd /home/circleci/app &&  npm install && npm test -- --watchAll=false"
          sudo chown -R `id -u`:`id -g` "$WORKSPACE"
          """

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
      
    stage('Deploy to Dev') {
     when {
         expression { ENVIRONMENT == 'dev' }
     }        
     steps{
        script {
          sh 'echo "Deploy to Dev"'
        }        }
      }  

    stage('Deploy to QA') {
     when {
         expression { ENVIRONMENT == 'qa' }
     }        
     steps{
        script {
          sh 'echo "Deploy to QA"'
        }        }
      }  

    stage('Deploy to Pre-Prod') {
     when {
         expression { ENVIRONMENT == 'pre-prod' }
     }        
     steps{
        script {
          sh 'echo "Deploy to Pre-Prod"'
        }        }
      }  

    stage('Deploy to Prod') {
     when {
         expression { ENVIRONMENT == 'prod' }
     }        
     steps{
        script {
          sh 'echo "Deploy to Prod"'
        }        }
      }                        
      
    }
}