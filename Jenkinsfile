pipeline {
agent any

```
environment {
    DOCKER_IMAGE = 'nageshjdevops/sprintboot-service'
    IMAGE_TAG = "${BUILD_NUMBER}"
    FULL_IMAGE = "${DOCKER_IMAGE}:${IMAGE_TAG}"
    MAVEN_HOME = '/mnt/build-tools/apache-maven-3.9.16'
    PATH = "/mnt/build-tools/apache-maven-3.9.16/bin:${env.PATH}"
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

    stage('Verify Tools') {
        steps {
            sh '''
                echo "===== Java ====="
                java -version

                echo "===== Maven ====="
                mvn -version

                echo "===== Docker ====="
                docker --version

                echo "===== Kubernetes ====="
                kubectl version --client
            '''
        }
    }

    stage('Build Application') {
        steps {
            echo 'Building Spring Boot application with Maven...'

            sh '''
                mvn clean package -DskipTests
            '''
        }
    }

    stage('Verify JAR') {
        steps {
            sh '''
                echo "===== Generated JAR ====="
                ls -lh target/*.jar
            '''
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
            echo "Pushing Docker image: ${FULL_IMAGE}"

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

    stage('Verify Kubernetes Deployment') {
        steps {
            echo 'Checking Kubernetes deployment...'

            sh '''
                echo "===== Nodes ====="
                kubectl get nodes

                echo "===== Deployment ====="
                kubectl get deployment ms1deploy

                echo "===== Pods ====="
                kubectl get pods -o wide

                echo "===== Service ====="
                kubectl get service ms1service

                echo "===== Running Image ====="
                kubectl get deployment ms1deploy \
                    -o jsonpath='{.spec.template.spec.containers[?(@.name=="m1")].image}'
                echo
            '''
        }
    }
}

post {

    success {
        echo "=========================================="
        echo "       CI/CD PIPELINE SUCCESSFUL"
        echo "=========================================="
        echo "Docker Image: ${FULL_IMAGE}"
        echo "Deployment: ms1deploy"
        echo "Container: m1"
    }

    failure {
        echo "=========================================="
        echo "         CI/CD PIPELINE FAILED"
        echo "=========================================="
    }

    always {
        echo "Pipeline completed. Build Number: ${BUILD_NUMBER}"
    }
}
```

}



