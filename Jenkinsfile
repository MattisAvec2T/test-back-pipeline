pipeline {
    agent { label 'docker-agent' }

    environment {
        REPOSITORY     = 'https://github.com/MattisAvec2T/test-back-pipeline.git'
        IMAGE_NAME     = 'backend-node'
        CONTAINER_NAME = 'backend-node'
        PORT           = '3000'
        NODE_IMAGE     = 'node:22.13.1-alpine3.21'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPOSITORY}"
            }
        }

        stage('Install') {
            steps {
                sh 'docker run --rm -v $(pwd):/app -w /app ${NODE_IMAGE} npm install'
            }
        }

        stage('Test') {
            // Remplacer le script "test" dans package.json par de vrais tests (ex: jest)
            steps {
                sh 'docker run --rm -v $(pwd):/app -w /app ${NODE_IMAGE} npm test'
            }
        }

        stage('Build image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
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
