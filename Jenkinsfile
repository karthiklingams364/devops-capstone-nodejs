pipeline {
    agent any

    environment {
        IMAGE = "karthiklingams364/node-app"
        APP_SERVER = "ubuntu@3.91.15.96"
        CONTAINER_NAME = "nodeapp"
    }

    stages {

        stage('Build') {
            steps {
                sh """
                docker build -t ${IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${APP_SERVER} '
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker pull ${IMAGE}:${BUILD_NUMBER}
                    docker run -d --name ${CONTAINER_NAME} -p 3001:3000 ${IMAGE}:${BUILD_NUMBER}
                '
                """
            }
        }
    }
}
