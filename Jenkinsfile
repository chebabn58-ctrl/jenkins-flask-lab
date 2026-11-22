
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-flask-app'
        CONTAINER_NAME = 'jenkins-flask-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm -e PYTHONPATH=/app $IMAGE_NAME:$BUILD_NUMBER pytest -v'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true
                    docker run -d \
                      --name $CONTAINER_NAME \
                      -p 8081:8000 \
                      -e APP_VERSION=$BUILD_NUMBER \
                      $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 5
                    docker exec $CONTAINER_NAME python -c "import urllib.request; assert urllib.request.urlopen('http://127.0.0.1:8000/health').status == 200"
                '''
            }
        }
    }
}
