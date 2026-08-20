pipeline {
    agent any

    environment {
        REGISTRY = '10.110.60.89:5000'
        IMAGE_NAME = 'my-docker-hosted/ci-cd-devops-lab'
        IMAGE = "${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
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
                      -t ${IMAGE} \
                      .
                '''
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASSWORD" | docker login ${REGISTRY} \
                          -u "$NEXUS_USERNAME" \
                          --password-stdin

                        docker push ${IMAGE}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying ${IMAGE}"

                    docker rm -f ci-cd-devops-app 2>/dev/null || true

                    docker pull ${IMAGE}

                    docker run -d \
                      --name ci-cd-devops-app \
                      -p 5051:5000 \
                      ${IMAGE}

                    sleep 3

                    docker ps --filter name=ci-cd-devops-app
                '''
            }
        }
    }

    post {
        always {
            sh '''
                rm -rf .docker-test-context

                docker image rm ${IMAGE} 2>/dev/null || true
                docker image rm ci-cd-devops-test:${BUILD_NUMBER} 2>/dev/null || true
            '''
        }
    }
}
