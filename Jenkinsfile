pipeline {
    agent { label 'linux-agent' }

    tools {
        maven 'Maven-3.9.16'
        jdk 'JDK21'
    }

    environment {
        APP_NAME = 'simple-java-app'
        DEPLOY_PATH = "/usr/share/nginx/html/${params.DEPLOY_ENV}"
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

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    mkdir -p ${DEPLOY_PATH}
                    cp index.html ${DEPLOY_PATH}/index.html
                    echo "Deployed to: ${DEPLOY_PATH}"
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline execution finished."
        }
        success {
            echo "Pipeline completed successfully! Code Quality Passed!"
        }
        failure {
            echo "Pipeline failed! Check logs or SonarQube Quality Gate."
        }
    }
}
