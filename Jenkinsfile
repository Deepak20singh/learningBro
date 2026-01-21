
pipeline {
    agent any

    environment {
        APP_NAME = "static-site"
        IMAGE_TAG = "latest"    // You can add BUILD_NUMBER for versioning
        CONTAINER_PORT = "80"
        HOST_PORT = "80"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Deepak20singh/learningBro.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t ${APP_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Stop Old Container') {
            steps {
                sh """
                  docker stop ${APP_NAME} || true
                  docker rm ${APP_NAME} || true
                """
            }
        }

        stage('Run New Container') {
            steps {
                sh """
                  docker run -d --name ${APP_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} \\
                    --restart unless-stopped \\
                    ${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Smoke Test') {
            steps {
                sh "curl -I http://localhost:${HOST_PORT} || (docker logs ${APP_NAME} && exit 1)"
            }
        }
    }

    post {
        always {
            sh "docker ps --format 'table {{.Names}}\\t{{.Image}}\\t{{.Status}}'"
        }
        failure {
            echo 'Deployment failed. Check Docker logs below:'
            sh "docker logs ${APP_NAME} || true"
        }
        success {
            echo "Deployed successfully at http://<EC2_PUBLIC_IP>/"
        }
    }
}
