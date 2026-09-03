pipeline {
    agent any

    stages {

        stage('Hello') {
            steps {
                echo 'Hello from Jenkins!'
            }
        }

        stage('Check Linux') {
            steps {
                sh 'whoami'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Check Git') {
            steps {
                sh 'git --version'
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker --version'
            }
        }
    }
}
