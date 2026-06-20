pipeline {

    agent any

    tools {
        nodejs 'node18'
    }

    environment {
        IMAGE_NAME = "swamybhagath02/lab-nodejs"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/SwamyBhagathEvuri/AWS-Lambda-01.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --legacy-peer-deps'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE_NAME:$TAG

                    docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest

                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                helm upgrade --install nodejs-app ./nodejs-app \
                --set image.repository=$IMAGE_NAME \
                --set image.tag=$TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check console logs.'
        }
    }
}
