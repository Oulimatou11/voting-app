pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        VERSION = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = "oulimatou"

        APP_VERSION = "v1.${BUILD_NUMBER}"
        
        VOTE_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-vote"
        RESULT_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-result"
        WORKER_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-worker"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-credentials', url: 'https://github.com/Oulimatou11/voting-app.git', branch: 'main'
            }
        }

        stage('Build Images') {
            steps {
                script {
                    echo "Building Docker images..."
                    sh "docker-compose build"

                    sh """
                        docker tag voting-app-pipeline-vote:latest ${VOTE_SERVICE}:${APP_VERSION}
                        docker tag voting-app-pipeline-result:latest ${RESULT_SERVICE}:${APP_VERSION}
                        docker tag voting-app-pipeline-worker:latest ${WORKER_SERVICE}:${APP_VERSION}
                    """
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    echo "Logging into DockerHub..."
                    sh "echo '${DOCKERHUB_CREDENTIALS_PSW}' | docker login -u '${DOCKERHUB_CREDENTIALS_USR}' --password-stdin"

                    echo "Pushing Docker images..."
                    sh """
                        docker push ${VOTE_SERVICE}:${APP_VERSION}
                        docker push ${RESULT_SERVICE}:${APP_VERSION}
                        docker push ${WORKER_SERVICE}:${APP_VERSION}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Updating docker-compose.yml with new image versions..."

                    sh """
                        sed -i '/vote:/!b;n;c\\    image: ${VOTE_SERVICE}:${APP_VERSION}' docker-compose.yml
			sed -i '/result:/!b;n;c\\    image: ${RESULT_SERVICE}:${APP_VERSION}' docker-compose.yml
			sed -i '/worker:/!b;n;c\\    image: ${WORKER_SERVICE}:${APP_VERSION}' docker-compose.yml
                    """

                    echo "Stopping existing containers..."
                    sh "docker-compose down || true"

                    echo "Starting new containers..."
                    sh "docker-compose up -d"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            echo "Cleaning up..."
            sh "docker logout"
        }
    }
}

