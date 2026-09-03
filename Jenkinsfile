pipeline {
    agent { label 'linux-agent' }

    tools {
        maven 'Maven-3.9.16'
        jdk 'JDK21'
    }

    environment {
        APP_NAME = 'simple-java-app'
        DOCKER_HUB_USER = 'shivamgupta898'
        IMAGE_NAME = "${DOCKER_HUB_USER}/simple-java-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DEPLOY_PATH = "/usr/share/nginx/html/${params.DEPLOY_ENV}"
        S3_BUCKET = 'shivam-maven-artifacts-builds-2026'
    }

    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'qa', 'prod'], description: 'Deployment Environment')
    }

    stages {
        stage('Environment Info') {
            steps {
                echo "Job Name: ${env.JOB_NAME}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "Application: ${APP_NAME}"
                echo "Environment: ${params.DEPLOY_ENV}"
            }
        }

        stage('Use Secret') {
            steps {
                withCredentials([string(credentialsId: 'demo-secret-text', variable: 'DEMO_SECRET')]) {
                    sh 'echo "Secret is: ${DEMO_SECRET}"'
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=simple-java-app'
                }
            }
        }

        /*
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        */

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        } 

        stage('Upload Artifact to S3') {
            steps {
                echo 'Uploading JAR artifact to S3 bucket: ${S3_BUCKET}...'
                sh """
                    aws s3 cp target/*.jar s3://${S3_BUCKET}/builds/build-${BUILD_NUMBER}/
                """
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest .
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop java-app-container || true
                    docker rm java-app-container || true
                    docker run -d --name java-app-container ${IMAGE_NAME}:latest
                    echo "Container deployed and running successfully!"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline execution finished."
            sh 'docker logout || true'
        }
        success {
            echo "Pipeline completed successfully! Docker image pushed & deployed!"
        }
        failure {
            echo "Pipeline failed! Check logs."
        }
    }
}
