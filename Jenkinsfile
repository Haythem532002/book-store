pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "haythem22/book-store"
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/Haythem532002/book-store.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'


            }
        }
        stage('Test') {
            steps {
                sh '. venv/bin/activate && python3 manage.py test'
            }
        }
        stage('Clean Old Docker Image') {
            steps {
                sh 'sudo docker rmi ${DOCKER_IMAGE}:latest || exit 0' 
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker stop book-store || exit 0'
                sh 'docker rm book-store || exit 0'
                sh "docker run -d -p 8000:8000 --name book-store ${DOCKER_IMAGE}:latest"
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
