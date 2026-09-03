pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'docker build -t jenkins-docker-app:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'docker run --rm jenkins-docker-app:${BUILD_NUMBER} python -m pytest'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f jenkins-docker-app || true

                    docker run -d \
                      --name jenkins-docker-app \
                      -p 5000:5000 \
                      jenkins-docker-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://localhost:5000/health
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
