pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'Maven 3'
    }

    environment {
        REMOTE_HOST = "docker-app-vm"   // alias SSH
        IMAGE_NAME  = "jenkins-app:latest"
        CONTAINER_NAME = "jenkins-app"
        APP_PORT = "9090"
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Check JDK') {
            steps {
                sh 'java -version'
                sh 'echo $JAVA_HOME'
            }
        }

        stage('Build Maven') {
            steps { sh 'mvn clean package' }
        }

        stage('Docker Build') {
            steps { sh "docker build -t $IMAGE_NAME ." }
        }

        stage('Check Docker on Remote') {
            steps {
                sh """
                ssh $REMOTE_HOST '
                    if ! command -v docker > /dev/null; then
                        echo "Docker is not installed on remote host!"
                        exit 1
                    fi
                    echo "Docker is installed ✅"
                '
                """
            }
        }

        stage('Push Image to Remote') {
            steps {
                sh """
                docker save $IMAGE_NAME | ssh $REMOTE_HOST 'docker load'
                """
            }
        }

        stage('Run Container Remote') {
            steps {
                sh """
                ssh $REMOTE_HOST '
                    # Stop & remove old container if exists
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true

                    # Run new container
                    docker run -d --name $CONTAINER_NAME -p $APP_PORT:$APP_PORT $IMAGE_NAME

                    # Wait a few seconds to ensure container is up
                    sleep 5

                    # Display running container status
                    docker ps | grep $CONTAINER_NAME

                    # Show last 10 logs
                    echo "Last 10 logs of $CONTAINER_NAME:"
                    docker logs --tail 10 $CONTAINER_NAME
                '
                """
            }
        }

        stage('Check Deployment') {
            steps {
                echo "App should be running at: http://$REMOTE_HOST:$APP_PORT/hello"
            }
        }
    }

    post {
        success { echo "Pipeline completed successfully! 🎉" }
        failure { echo "Pipeline failed. Check logs! ⚠️" }
    }
}
