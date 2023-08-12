pipeline {
    agent any

    environment {
        // Initialize ENVIRONMENT with a default value, but allow it to be overridden by the parameter
        ENVIRONMENT = "${ENVIRONMENT}"
        NODE_IMAGE = "circleci/node:16.13.1-bullseye-browsers"
        SERVICE_NAME = "test-repo-delete" 
        ECR_ADDRESS = "064827688814.dkr.ecr.eu-west-3.amazonaws.com"
        IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-${ENVIRONMENT}"
        IMAGE_TAG = "${BUILD_NUMBER}"
        JOB_BUILD_NUMBER = "${BUILD_NUMBER}"

    }

   
    stages {

    stage('Initialization') {
      steps{
        script {
            sh 'echo "Environment     =  $ENVIRONMENT"'
            sh 'echo "SERVICE_NAME    =  $SERVICE_NAME"'
            sh 'echo "REPO_NAME       =  $REPO_NAME"'
            sh 'echo "ECR_ADDRESS       =  $ECR_ADDRESS"'
            sh 'echo "IMAGE_NAME       =  $IMAGE_NAME"'
            sh 'echo "IMAGE_TAG       =  $IMAGE_TAG"'

        }
      }
    }

    stage('Unit Tests') {
      steps{
        script {
          sh 'echo "In Unit Tests $ENVIRONMENT"'
          sh 'echo "Running using"; whoami'
          sh """
          docker run --rm --user root -v "$WORKSPACE":/home/circleci/app $NODE_IMAGE /bin/bash -c "cd /home/circleci/app &&  npm install && npm test -- --watchAll=false"
          """

        }
      }
    }
        
    // Building Docker images
    stage('Building image') {
      steps{
        script {
            if (env.ENVIRONMENT == 'dev') {
                sh 'echo "Build Image for $ENVIRONMENT Environment"'
                //sh 'docker build -t test-repo-delete:latest .'
                //sh 'docker tag test-repo-delete:latest $IMAGE_NAME:$IMAGE_TAG'

                sh 'echo "docker build $SERVICE_NAME:latest"'
                sh 'echo "docker tag $SERVICE_NAME:latest $IMAGE_NAME:latest"'
                sh 'echo "docker tag $SERVICE_NAME:latest $IMAGE_NAME:$JOB_BUILD_NUMBER"'
            }
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
        script {
                sh 'echo "Pushing to ECR"'
                sh 'echo "docker push $IMAGE_NAME:latest"'
                sh 'echo "docker push $IMAGE_NAME:$JOB_BUILD_NUMBER"'          
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
                sh 'echo "docker run $IMAGE_NAME:$JOB_BUILD_NUMBER"'       

        }       
     }
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