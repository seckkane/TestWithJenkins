pipeline {
    agent any

    tools {
        jdk 'JDK 17'          // doit correspondre à ton JDK configuré dans Global Tool Configuration
        maven 'Maven 3'   // si tu as configuré Maven ici
    }

    environment {
        REMOTE_HOST = "192.168.56.20"
        REMOTE_USER = "vagrant"
        IMAGE_NAME  = "jenkins-app:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check JDK') {
            steps {
                sh 'java -version'
                sh 'echo $JAVA_HOME'
            }
        }

        stage('Build Maven') {
            steps {
                sh '''
                mvn clean package
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Push Image to Remote') {
            steps {
                sh '''
                # Transfert de l'image vers docker-app-vm via SSH
                docker save $IMAGE_NAME | ssh $REMOTE_USER@$REMOTE_HOST "docker load"
                '''
            }
        }

        stage('Run Container Remote') {
            steps {
                sh '''
                ssh $REMOTE_USER@$REMOTE_HOST "
                    docker stop jenkins-app || true
                    docker rm jenkins-app || true
                    docker run -d --name jenkins-app -p 9090:9090 $IMAGE_NAME
                "
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                echo "App should now be running on docker-app-vm: http://$REMOTE_HOST:9090/hello"
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully! 🎉"
        }
        failure {
            echo "Pipeline failed. Check logs! ⚠️"
        }
    }
}
