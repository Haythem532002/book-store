pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "haythem22/book-store:latest"
        IMAGE_TAG = "latest"
        MINIKUBE_HOME = "/home/haythem"
        AZURE_RESOURCE_GROUP = "book-store-ressource"
        AZURE_AKS_CLUSTER_NAME = "book-store-aks"
        AZURE_TENANT_ID = "dbd6664d-4eb9-46eb-99d8-5c43ba153c61" 
        SONARQUBE_ENV = "sonar-q"
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
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-q') {
                    script {
                        def scannerHome = tool 'SonarQubeScanner-7.1.0'
                        sh "${scannerHome}/bin/sonar-scanner 
                            -Dsonar.projectKey=book-store 
                            -Dsonar.sources=. 
                            -Dsonar.python.version=3 
                            -Dsonar.sourceEncoding=UTF-8"
                    }
                }
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

        stage('Login to Azure') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-service-principal', usernameVariable: 'AZURE_CLIENT_ID', passwordVariable: 'AZURE_CLIENT_SECRET')]) {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID'
                }
            }
        }

        stage('Get AKS Credentials') {
            steps {
                sh 'az aks get-credentials --resource-group $AZURE_RESOURCE_GROUP --name $AZURE_AKS_CLUSTER_NAME --overwrite-existing'
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh 'kubectl apply -f book-store-deploy.yaml'
                sh 'kubectl apply -f book-store-service.yaml'
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
