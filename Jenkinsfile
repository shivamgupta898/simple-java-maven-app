pipeline {
    agent any 

    environment {
        APP_NAME = 'simple-java-app'
        DEPLOY_ENV = 'dev'
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
