pipeline {
    agent any 

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'stage', 'prod'],
            description: 'Select deployment environment'
        )
    }

    environment {
        APP_NAME = 'simple-java-app'
        DEPLOY_PATH = '/usr/share/nginx/html' 
    }

    stages { 
        
        stage('Environment Info') {
            steps {
                sh '''
                   echo "Job Name: $JOB_NAME"
                   echo "Build Number: $BUILD_NUMBER"
                   echo "Workspace: $WORKSPACE"
                   echo "Application: $APP_NAME"
                   echo "Environment: $DEPLOY_ENV"
                   echo "Deploy Path: $DEPLOY_PATH"
                   ''' 
            } 
        } 

        stage('Use Secret') {
            steps {
                withCredentials([string(credentialsId: 'demo-secret', variable: 'DEMO_SECRET')]) {
                    sh '''
                       echo "Secret is: $DEMO_SECRET"
                       '''
                }
            }
        } 
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                sh 'cp index.html $DEPLOY_PATH/index.html'
            }
        }
    }
}
