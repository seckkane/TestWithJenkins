pipeline {
    agent any

    tools {
        maven 'Maven 3' // le nom que tu as configuré dans Global Tool Configuration
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t jenkins-app:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker stop jenkins-app || true
                docker rm jenkins-app || true
                docker run -d --name jenkins-app -p 9090:9090 jenkins-app:latest
                '''
            }
        }
    }
}
