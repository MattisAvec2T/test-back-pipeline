pipeline {
    agent { label 'docker-agent' }

    environment {
        REPOSITORY     = 'https://github.com/MattisAvec2T/test-back-pipeline.git'
        IMAGE_NAME     = 'test-pipeline-back'
        CONTAINER_NAME = 'test-pipeline-back'
        PORT           = '3000'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPOSITORY}"
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push on DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh """
                        echo "\$DOCKERHUB_TOKEN" | docker login -u "\$DOCKERHUB_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \$DOCKERHUB_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker tag ${IMAGE_NAME}:latest \$DOCKERHUB_USER/${IMAGE_NAME}:latest
                        docker push \$DOCKERHUB_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push \$DOCKERHUB_USER/${IMAGE_NAME}:latest
                        docker logout
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "docker rm -f ${CONTAINER_NAME} || true"
                sh "docker run -d --name ${CONTAINER_NAME} --network devops -p ${PORT}:${PORT} ${IMAGE_NAME}:latest"
            }
        }

    }

    post {
        success {
            echo "Backend disponible sur http://localhost:${PORT}"
        }
        failure {
            echo 'Le pipeline a échoué — vérifier les logs ci-dessus.'
        }
    }
}
