pipeline {
    agent any

    environment {
        IMAGE = "YOUR_DOCKERHUB_USERNAME/node-app"
        APP_SERVER = "APP_SERVER_IP"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/YOUR_USERNAME/devops-capstone-nodejs.git'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE:$BUILD_NUMBER .'
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
                docker push $IMAGE:$BUILD_NUMBER
                '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh ubuntu@$APP_SERVER '
                docker stop nodeapp || true
                docker rm nodeapp || true
                docker run -d --name nodeapp -p 3001:3000 $IMAGE:$BUILD_NUMBER
                '
                """
            }
        }
    }
}
