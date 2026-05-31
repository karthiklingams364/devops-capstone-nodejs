pipeline {
    agent any

    environment {
        IMAGE = "karthiklingams364/node-app"
        APP_SERVER = "3.91.15.96"
        CONTAINER_NAME = "nodeapp"
        DOCKER = "/usr/bin/docker"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                echo "Checking Docker..."
                which docker || true
                /usr/bin/docker --version
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                ${DOCKER} build -t ${IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                    echo $PASS | ${DOCKER} login -u $USER --password-stdin
                    ${DOCKER} push ${IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} '
                    docker stop ${CONTAINER_NAME} || true &&
                    docker rm ${CONTAINER_NAME} || true &&
                    docker pull ${IMAGE}:${BUILD_NUMBER} &&
                    docker run -d --name ${CONTAINER_NAME} -p 3001:3000 ${IMAGE}:${BUILD_NUMBER}
                '
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Successful!"
        }
        failure {
            echo "❌ Pipeline Failed - Check logs"
        }
    }
}
