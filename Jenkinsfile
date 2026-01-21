
pipeline {
    agent any

    environment {
        // ---- App & Image Config ----
        APP_NAME       = "static-site"                 // container name
        IMAGE_TAG      = "build-${BUILD_NUMBER}"       // versioned tag for each run

        // ---- Ports ----
        CONTAINER_PORT = "80"                          // nginx inside container
        HOST_PORT      = "80"                          // change to "8080" if 80 is busy

        // ---- Git Repo ----
        REPO_URL       = "https://github.com/Deepak20singh/learningBro.git"
        BRANCH_NAME    = "main"
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t ${APP_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Stop & Remove Old Container') {
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
                  docker run -d --name ${APP_NAME} \
                    -p ${HOST_PORT}:${CONTAINER_PORT} \
                    --restart unless-stopped \
                    ${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Smoke Test') {
            steps {
                sh "curl -I http://localhost:${HOST_PORT} || (docker logs ${APP_NAME} && exit 1)"
            }
        }

        stage('Cleanup Old Images') {
            steps {
                // remove dangling layers to save disk
                sh "docker image prune -f || true"
            }
        }
    }

    post {
        always {
            sh "docker ps --format 'table {{.Names}}\\t{{.Image}}\\t{{.Status}}'"
        }
        failure {
            echo '❌ Deployment failed. Docker logs:'
            sh "docker logs ${APP_NAME} || true"
        }
        success {
            echo "✅ Deployment successful on instance: jenkins"
            echo "👉 Open: http://<EC2_PUBLIC_IP>${HOST_PORT == '80' ? '' : (':' + HOST_PORT)}/"
        }
    }
}


