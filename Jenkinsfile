pipeline{
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        VERSION = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = "sene.oulimatou@gmail.com"

        APP_VERSION = "v1.${BUILD_NUMBER}"  
        
        VOTE_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-vote"
        RESULT_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-result"
        WORKER_SERVICE = "${DOCKERHUB_USERNAME}/vote-app-worker"
    }

    stage('Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/Oulimatou11/voting-app.git'
        }
    }

    stage('Build Images') {
        steps {
            script {
                
                sh "docker-compose build"
                    
                sh """
                    docker tag voting-app-vote:latest ${VOTE_SERVICE}:${APP_VERSION}
                    docker tag voting-app-result:latest ${RESULT_SERVICE}:${APP_VERSION}
                    docker tag voting-app-worker:latest ${WORKER_SERVICE}:${APP_VERSION}
                    """
                    
                echo "Built Successfully"
            }
        }
    }

    stage('Push Images') {
        steps {
            script {
                
                sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    
        
                sh """
                    docker push ${VOTE_SERVICE}:${APP_VERSION}
                    docker push ${RESULT_SERVICE}:${APP_VERSION}
                    docker push ${WORKER_SERVICE}:${APP_VERSION}
                """
                    
                echo "Publish Successfully"
            }
        }
    }

    stage('Deploy') {
        steps {
            script {
                
                sh """
                    sed -i 's|build:|image: ${VOTE_SERVICE}:${APP_VERSION}|' docker-compose.yml
                    sed -i 's|build:|image: ${RESULT_SERVICE}:${APP_VERSION}|' docker-compose.yml
                    sed -i 's|build:|image: ${WORKER_SERVICE}:${APP_VERSION}|' docker-compose.yml
                """
            
                sh "docker-compose down || true"
                    
                sh "docker-compose up -d"
                    
                echo "Deployed Successfully"
            }
        }
    }

    post {
        success {
            echo "Pipeline execute successfully ! "
        }
        failure {
            echo "Pipeline failure"
        }
        always {
            echo " Disconnecting..."
            sh "docker logout"
        }
    }

}
