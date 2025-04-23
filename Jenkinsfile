pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/django-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sanjuumahabal/BookRentalSystem.git',
                credentialsId: 'github-credentials'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'python -m venv venv'
                bat 'call venv\\Scripts\\activate && pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'call venv\\Scripts\\activate && python manage.py test'
            }
        }

        stage('Cleanup Old Docker Image') {
            steps {
                bat 'docker rmi ${DOCKER_IMAGE}:latest || exit 0'  // Removes old image if it exists
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker stop django-app || exit 0'
                bat 'docker rm django-app || exit 0'
                bat "docker run -d -p 8000:8000 --name django-app ${DOCKER_IMAGE}:latest"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
