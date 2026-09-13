pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'nageshjdevops/sprintboot-service'
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${DOCKER_IMAGE}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/nageshjdevops/SprintBootService-1.git'
                    ]]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${FULL_IMAGE}"

                sh 'docker build -t ${FULL_IMAGE} .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${FULL_IMAGE}'
            }
        }

        stage('Docker Logout') {
            steps {
                sh 'docker logout || true'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/ms1deploy \
                        m1=${FULL_IMAGE}

                    kubectl rollout status deployment/ms1deploy \
                        --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== Deployment ====="
                    kubectl get deployment ms1deploy

                    echo "===== Pods ====="
                    kubectl get pods -o wide

                    echo "===== Service ====="
                    kubectl get service ms1service

                    echo "===== Image ====="
                    kubectl get deployment ms1deploy \
                        -o jsonpath='{.spec.template.spec.containers[?(@.name=="m1")].image}'
                    echo
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD PIPELINE SUCCESSFUL"
            echo "Docker Image: ${FULL_IMAGE}"
        }

        failure {
            echo "CI/CD PIPELINE FAILED"
        }

        always {
            echo "Build Number: ${BUILD_NUMBER}"
        }
    }
}


