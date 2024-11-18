pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-nginx-site'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Howard-sa-git/Jenkins-Docker-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker rm -f my-nginx-container || true'
                sh 'docker run -d -p 8090:80 --name my-nginx-container ${DOCKER_IMAGE}'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
            sh 'docker rm -f my-nginx-container || true'
        }
    }
}
