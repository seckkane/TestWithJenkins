pipeline {
    agent any

    environment {
        // Définir JAVA_HOME si nécessaire
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checkout du code depuis Git"
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
                echo "Compilation et package avec Maven"
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Construction de l\'image Docker"
                sh 'docker build -t jenkins-app:latest .'
            }
        }

        stage('Push Image to Remote (optional)') {
            steps {
                echo "Optionnel : envoyer l\'image à docker-app-vm via SSH"
                // Si on veut pousser l'image directement, sinon on peut utiliser docker save/load
                sh '''
                docker save jenkins-app:latest | ssh vagrant@192.168.56.20 "docker load"
                '''
            }
        }

        stage('Run Container Remote') {
            steps {
                echo "Déploiement à distance sur docker-app-vm"
                sh '''
                ssh vagrant@192.168.56.20 "
                    docker stop jenkins-app || true
                    docker rm jenkins-app || true
                    docker run -d --name jenkins-app -p 9090:9090 jenkins-app:latest
                "
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline terminé avec succès !"
        }
        failure {
            echo "Pipeline échoué 😢"
        }
    }
}
