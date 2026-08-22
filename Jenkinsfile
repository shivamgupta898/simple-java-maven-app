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
                '''
            }
        }

        stage('Use Secret') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'demo-secret',
                        variable: 'DEMO_SECRET'
                    )
                ]) {
                    sh '''
                        echo "Secret is: $DEMO_SECRET"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    DEPLOY_PATH="/usr/share/nginx/html/$DEPLOY_ENV"

                    mkdir -p "$DEPLOY_PATH"
                    cp index.html "$DEPLOY_PATH/index.html"

                    echo "Deployed to: $DEPLOY_PATH"
                '''
            }
        }
    }
}
