pipeline {
    agent any

    environment {
        // Initialize ENVIRONMENT with a default value, but allow it to be overridden by the parameter
        ENVIRONMENT = "${ENVIRONMENT}"
        NODE_IMAGE = "circleci/node:16.13.1-bullseye-browsers"
        SERVICE_NAME = "test-repo-delete" 
        ECR_ADDRESS = "064827688814.dkr.ecr.eu-west-3.amazonaws.com"
        IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-${ENVIRONMENT}"
        DEV_IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-dev"
        QA_IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-qa"
        PREPROD_IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-pre-prod"
        PROD_IMAGE_NAME = "${ECR_ADDRESS}/${SERVICE_NAME}-prod"

        IMAGE_TAG = "${BUILD_NUMBER}"
        JOB_BUILD_NUMBER = "${BUILD_NUMBER}"

    }

   
    stages {


        stage('Check Version Change') {
            steps {
                script {
                    env.PREVIOUS_VERSION = sh(script: 'git show HEAD~1:version.txt | grep service_version', returnStdout: true).trim().split('=')[1].trim()
                    env.CURRENT_VERSION = readFile('version.txt').trim().split('=')[1].trim()

                    if (env.PREVIOUS_VERSION == env.CURRENT_VERSION) {
                        echo "No change in version.txt, using build number"
                        echo "previousVersion = $env.PREVIOUS_VERSION currentVersion=$env.CURRENT_VERSION"
                        //env.IMAGE_TAG = JOB_BUILD_NUMBER
                    } else {
                        echo "Version.txt changed, using version from version.txt"
                        echo "previousVersion = $env.PREVIOUS_VERSION currentVersion=$env.CURRENT_VERSION"
                        //env.IMAGE_TAG = currentVersion
                    }
                }
            }
        }


        stage('Initialization') {
        steps{
            script {
                sh 'echo "Environment     =  $ENVIRONMENT"'
                sh 'echo "SERVICE_NAME    =  $SERVICE_NAME"'
                sh 'echo "REPO_NAME       =  $REPO_NAME"'
                sh 'echo "ECR_ADDRESS       =  $ECR_ADDRESS"'
                sh 'echo "IMAGE_NAME       =  $IMAGE_NAME"'
                sh 'echo "IMAGE_TAG       =  $IMAGE_TAG"'
                
                def versionLine = sh(script: 'cat version.txt | grep service_version', returnStdout: true).trim()
                def version = versionLine.split('=')[1].trim()
                
                echo "Version from version.txt: $version"
                env.SERVICE_VERSION = version
                echo "SERVICE_VERSION = $SERVICE_VERSION"

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
            when {
                expression { ENVIRONMENT == 'dev' }
            } 

            steps{
                script {
                    if (env.ENVIRONMENT == 'dev') {
                        sh 'echo "Build Image for $ENVIRONMENT Environment"'                         
                        sh 'docker build -t $SERVICE_NAME:latest .'
                        sh 'docker tag $SERVICE_NAME:latest $IMAGE_NAME:latest'
                        sh 'docker tag $SERVICE_NAME:latest $IMAGE_NAME:$JOB_BUILD_NUMBER'

                        if (env.PREVIOUS_VERSION == env.CURRENT_VERSION) {
                            echo "Version.txt changed, using version from version.txt as well"
                            echo "previousVersion = $env.PREVIOUS_VERSION currentVersion=$env.CURRENT_VERSION"
                            sh 'docker tag $SERVICE_NAME:latest $IMAGE_NAME:$env.CURRENT_VERSION' 
                        }

                    }
                }
            }
        }
    
        stage('Pushing to ECR') {
            when {
                expression { ENVIRONMENT == 'dev' }
            } 

            steps{  
                script {
                        
                        withCredentials([usernamePassword(credentialsId: 'ecr-dev', passwordVariable: 'secret_key', usernameVariable: 'access_key')]) {
                            def awsLoginCmd = "aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${ECR_ADDRESS}"
                            def dockerLoginCmd = "AWS_ACCESS_KEY_ID=$access_key AWS_SECRET_ACCESS_KEY=$secret_key $awsLoginCmd"
                            sh "eval $dockerLoginCmd"
                            
                        }
                        sh 'docker push $IMAGE_NAME:latest'
                        sh 'docker push $IMAGE_NAME:$JOB_BUILD_NUMBER'
                        if (env.PREVIOUS_VERSION == env.CURRENT_VERSION) {
                            sh 'docker push $IMAGE_NAME:$env.CURRENT_VERSION'
                        }


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
                        sh 'echo "docker run $IMAGE_NAME:latest"'  
                        withCredentials([usernamePassword(credentialsId: 'ecr-dev', passwordVariable: 'secret_key', usernameVariable: 'access_key')]) {
                            def awsLoginCmd = "aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${ECR_ADDRESS}"
                            def dockerLoginCmd = "AWS_ACCESS_KEY_ID=$access_key AWS_SECRET_ACCESS_KEY=$secret_key $awsLoginCmd"
                            sh "eval $dockerLoginCmd"
                        }                     

                }       
            }
        }  


        stage('Promotion to Higher Environment') {
            when {
                expression { ENVIRONMENT in ['qa', 'pre-prod', 'prod'] }
            } 

            steps{  
                script {
                    if (ENVIRONMENT == 'qa' ) {

                        withCredentials([usernamePassword(credentialsId: 'ecr-dev', passwordVariable: 'secret_key', usernameVariable: 'access_key')]) {
                            def awsLoginCmd = "aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${ECR_ADDRESS}"
                            def dockerLoginCmd = "AWS_ACCESS_KEY_ID=$access_key AWS_SECRET_ACCESS_KEY=$secret_key $awsLoginCmd"
                            sh "eval $dockerLoginCmd"
                            
                        }

                        sh 'docker pull $DEV_IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker tag $DEV_IMAGE_NAME:$SERVICE_VERSION $IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker push $IMAGE_NAME:$SERVICE_VERSION'
                    }

                    if (ENVIRONMENT == 'pre-prod' ) {

                        withCredentials([usernamePassword(credentialsId: 'ecr-dev', passwordVariable: 'secret_key', usernameVariable: 'access_key')]) {
                            def awsLoginCmd = "aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${ECR_ADDRESS}"
                            def dockerLoginCmd = "AWS_ACCESS_KEY_ID=$access_key AWS_SECRET_ACCESS_KEY=$secret_key $awsLoginCmd"
                            sh "eval $dockerLoginCmd"
                            
                        }

                        sh 'docker pull $QA_IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker tag $QA_IMAGE_NAME:$SERVICE_VERSION $IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker push $IMAGE_NAME:$SERVICE_VERSION'
                    }

                    if (ENVIRONMENT == 'prod' ) {

                        withCredentials([usernamePassword(credentialsId: 'ecr-dev', passwordVariable: 'secret_key', usernameVariable: 'access_key')]) {
                            def awsLoginCmd = "aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${ECR_ADDRESS}"
                            def dockerLoginCmd = "AWS_ACCESS_KEY_ID=$access_key AWS_SECRET_ACCESS_KEY=$secret_key $awsLoginCmd"
                            sh "eval $dockerLoginCmd"
                            
                        }

                        sh 'docker pull $PREPROD_IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker tag $PREPROD_IMAGE_NAME:$SERVICE_VERSION $IMAGE_NAME:$SERVICE_VERSION'
                        sh 'docker push $IMAGE_NAME:$SERVICE_VERSION'
                    }                    

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
                sh 'echo "docker run $IMAGE_NAME:$SERVICE_VERSION"'  
                }        
            }
        }  

        stage('Deploy to Pre-Prod') {
            when {
                expression { ENVIRONMENT == 'pre-prod' }
            }        
            steps{
                script {
                sh 'echo "Deploy to Pre-Prod"'
                sh 'echo "docker run $IMAGE_NAME:$SERVICE_VERSION"'  
                }        
            }
        }  

        stage('Deploy to Prod') {
            when {
                expression { ENVIRONMENT == 'prod' }
            }        
            steps{
                script {
                sh 'echo "Deploy to Prod"'
                sh 'echo "docker run $IMAGE_NAME:$SERVICE_VERSION"'  
                }        
                }
        }                        
      
    }

}