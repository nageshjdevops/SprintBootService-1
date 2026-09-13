```groovy
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

                git(
                    branch: 'master',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/nageshjdevops/SprintBootService-1.git'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${FULL_IMAGE}"

                sh '''
                    docker build \
                        -t ${FULL_IMAGE} \
                        .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging into Docker Hub...'

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
                echo "Pushing Docker image: ${FULL_IMAGE}"

                sh '''
                    docker push ${FULL_IMAGE}
                '''
            }
        }

        stage('Docker Logout') {
            steps {
                sh '''
                    docker logout
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying ${FULL_IMAGE} to Kubernetes..."

                sh '''
                    kubectl set image deployment/ms1deploy \
                        m1=${FULL_IMAGE}

                    kubectl rollout status deployment/ms1deploy \
                        --timeout=180s
                '''
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                echo 'Checking Kubernetes deployment...'

                sh '''
                    echo "===== Deployment ====="
                    kubectl get deployment ms1deploy

                    echo "===== Pods ====="
                    kubectl get pods -o wide

                    echo "===== Service ====="
                    kubectl get service ms1service
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo " CI/CD PIPELINE SUCCESSFUL"
            echo " Image: ${FULL_IMAGE}"
            echo " Deployment: ms1deploy"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " CI/CD PIPELINE FAILED"
            echo "======================================"
        }

        always {
            echo "Pipeline finished. Build number: ${BUILD_NUMBER}"
        }
    }
}
```

