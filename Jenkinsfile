pipeline {
    agent any

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

        stage('Push Image to Remote') {
            steps {
                sh '''
                # On envoie directement l'image au serveur distant via SSH
                docker save jenkins-app:latest | ssh vagrant@192.168.56.20 "docker load"
                '''
            }
        }

        stage('Run Container Remote') {
            steps {
                sh '''
                ssh vagrant@192.168.56.20 "
                    docker stop jenkins-app || true
                    docker rm jenkins-app || true
                    docker run -d --name jenkins-app -p 9090:9090 jenkins-app:latest
                "
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                echo "App should now be running on docker-app-vm: http://192.168.56.20:9090/hello"
            }
        }

    }
}
