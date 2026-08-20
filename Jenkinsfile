pipeline {
    agent any

    environment {
        NEXUS_REGISTRY = '10.110.60.89:5000'
        NEXUS_REPOSITORY = 'my-docker-hosted'
        IMAGE_NAME = 'ci-cd-devops-lab'
        IMAGE = "${NEXUS_REGISTRY}/${NEXUS_REPOSITORY}/${IMAGE_NAME}:${BUILD_NUMBER}"
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

                    docker run --rm ci-cd-devops-test:${BUILD_NUMBER}
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building ${IMAGE}"

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
                        echo "${NEXUS_PASSWORD}" | docker login ${NEXUS_REGISTRY} \
                            -u "${NEXUS_USERNAME}" \
                            --password-stdin

                        docker push ${IMAGE}

                        docker logout ${NEXUS_REGISTRY}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying ${IMAGE} to Kubernetes"

                    kubectl set image deployment/ci-cd-devops-app \
                        ci-cd-devops-app=${IMAGE}

                    kubectl rollout status deployment/ci-cd-devops-app \
                        --timeout=120s

                    echo "Deployment successful"

                    kubectl get deployment ci-cd-devops-app
                    kubectl get pods -l app=ci-cd-devops-app
                '''
            }
        }
    }

    post {
        always {
            sh '''
                rm -rf .docker-test-context || true
                docker image rm ${IMAGE} || true
                docker image rm ci-cd-devops-test:${BUILD_NUMBER} || true
            '''
        }

        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed.'
        }
    }
}
