pipeline {
    agent any

    environment {
        IMAGE = "karthiklingams364/node-app"
        APP_SERVER = "3.91.15.96"
        CONTAINER_NAME = "nodeapp"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE}:${BUILD_NUMBER} ."
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
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${IMAGE}:${BUILD_NUMBER}
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
