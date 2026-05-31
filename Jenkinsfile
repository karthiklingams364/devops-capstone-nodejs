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

        stage('Build Image') {
            steps {
                sh "/usr/bin/docker build -t ${IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | /usr/bin/docker login -u $USER --password-stdin
                    /usr/bin/docker push ${IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@3.91.15.96 "
                    docker stop nodeapp || true &&
                    docker rm nodeapp || true &&
                    docker pull ${IMAGE}:${BUILD_NUMBER} &&
                    docker run -d --name nodeapp -p 3001:3000 ${IMAGE}:${BUILD_NUMBER}
                "
                '''
            }
        }
    }
}
