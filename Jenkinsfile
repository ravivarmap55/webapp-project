
pipeline {
    agent any

    environment {
        IMAGE = "varma2004/devops-app"
        TAG = "${BUILD_NUMBER}"
        APP_IP = "15.134.91.66"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    python -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t $IMAGE:$TAG \
                    -t $IMAGE:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE:$TAG
                        docker push $IMAGE:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to App EC2') {
            steps {
                sshagent(['app-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@$APP_IP "
                            docker pull $IMAGE:$TAG &&
                            (docker rm -f webapp || true) &&
                            docker run -d \
                            --name webapp \
                            --restart unless-stopped \
                            -p 80:5000 \
                            $IMAGE:$TAG
                        "
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    curl --fail http://$APP_IP/health
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
