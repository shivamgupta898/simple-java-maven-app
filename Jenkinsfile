pipeline {
    agent any

    stages { 
        
        stage('Environment Info') {
            steps {
                sh '''
                   echo "Job Name: $JOB_NAME"
                   echo "Build Number: $BUILD_NUMBER"
                   echo "Workspace: $WORKSPACE"
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
                sh 'cp index.html /usr/share/nginx/html/index.html'
            }
        }
    }
}
