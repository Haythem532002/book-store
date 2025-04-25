pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "haythem22/book-store"
        IMAGE_TAG = "latest"
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
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'haythem22-dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f book-store-deploy.yaml'
                sh 'kubectl apply -f book-store-service.yaml'
                sh 'echo "Link of the app : " && minikube service book-store --url'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            sh 'docker logout'
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
