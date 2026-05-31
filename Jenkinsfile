pipeline {
    agent any

    environment {
        IMAGE = "karthiklingams364/node-app"
        APP_SERVER = "3.91.15.96"
        CONTAINER_NAME = "nodeapp"
        PATH = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fix Docker PATH') {
            steps {
                sh '''
                export PATH=$PATH:/usr/bin:/usr/local/bin
                docker --version
                docker ps
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                export PATH=$PATH:/usr/bin:/usr/local/bin
                docker build -t ${IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    export PATH=$PATH:/usr/bin:/usr/local/bin
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
