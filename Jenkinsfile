pipeline {
    agent any

    environment {
        NEXUS_REGISTRY = '172.17.0.1:5000'
        IMAGE_NAME = 'my-docker-hosted/ci-cd-devops-lab'
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

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                      .
                '''
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-docker',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASSWORD" | docker login ${NEXUS_REGISTRY} \
                          -u "$NEXUS_USERNAME" \
                          --password-stdin

                        docker push \
                          ${NEXUS_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker logout ${NEXUS_REGISTRY}
                    '''
                }
            }
        }
    }

    post {
        always {
            sh '''
                rm -rf .docker-test-context

                docker image rm \
                  ${NEXUS_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                  2>/dev/null || true

                docker image rm \
                  ci-cd-devops-test:${BUILD_NUMBER} \
                  2>/dev/null || true
            '''
        }
    }
}
