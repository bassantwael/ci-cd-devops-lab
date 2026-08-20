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
                    rm -rf .docker-test-context

                    mkdir -p .docker-test-context

                    cp requirements.txt .docker-test-context/
                    cp pytest.ini .docker-test-context/
                    cp Dockerfile.test .docker-test-context/
                    cp -r app .docker-test-context/
                    cp -r tests .docker-test-context/

                    docker build \
                      -t ci-cd-devops-test:${BUILD_NUMBER} \
                      -f .docker-test-context/Dockerfile.test \
                      .docker-test-context

                    docker run --rm \
                      ci-cd-devops-test:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -rf .docker-test-context'
        }
    }
}
