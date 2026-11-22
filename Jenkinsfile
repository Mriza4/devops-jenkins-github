
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'rizamohamed456/devops-app'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKER_IMAGE:$BUILD_NUMBER
                        docker push $DOCKER_IMAGE:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on Same EC2') {
            steps {
                sh '''
                    docker pull $DOCKER_IMAGE:$BUILD_NUMBER

                    docker rm -f devops-app || true

                    docker run -d \
                    --restart unless-stopped \
                    --name devops-app \
                    -p 8081:80 \
                    $DOCKER_IMAGE:$BUILD_NUMBER

                    docker image prune -f
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed on Jenkins EC2!'
        }

        failure {
            echo 'FAILED: Check Jenkins console output.'
        }
    }
}
