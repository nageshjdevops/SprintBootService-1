pipeline {
agent any

```
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

            sh '''
                docker build -t ${FULL_IMAGE} .
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
            echo "Pushing ${FULL_IMAGE} to Docker Hub..."

            sh '''
                docker push ${FULL_IMAGE}
            '''
        }
    }

    stage('Docker Logout') {
        steps {
            sh '''
                docker logout || true
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

    stage('Verify Deployment') {
        steps {
            echo 'Verifying Kubernetes deployment...'

            sh '''
                echo "===== Deployment ====="
                kubectl get deployment ms1deploy

                echo "===== Pods ====="
                kubectl get pods -o wide

                echo "===== Service ====="
                kubectl get service ms1service

                echo "===== Current Image ====="
                kubectl get deployment ms1deploy \
                    -o jsonpath='{.spec.template.spec.containers[?(@.name=="m1")].image}'
                echo
            '''
        }
    }
}

post {
    success {
        echo "========================================"
        echo "       CI/CD PIPELINE SUCCESSFUL"
        echo "========================================"
        echo "Docker Image: ${FULL_IMAGE}"
        echo "Kubernetes Deployment: ms1deploy"
        echo "Kubernetes Container: m1"
    }

    failure {
        echo "========================================"
        echo "         CI/CD PIPELINE FAILED"
        echo "========================================"
    }

    always {
        echo "Pipeline completed. Build Number: ${BUILD_NUMBER}"
    }
}
```

}


