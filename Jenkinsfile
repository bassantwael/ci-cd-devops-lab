pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$PWD":/workspace \
                      -w /workspace \
                      python:3.10-slim \
                      bash -c "pip install --no-cache-dir -r requirements.txt && pytest -v"
                '''
            }
        }
    }
}
