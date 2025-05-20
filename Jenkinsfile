pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "botunga"
        DOCKERHUB_PASSWORD = "gangster16"
        IMAGE_NAME = "botunga/node-kube-demo"
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/b-otunga/node-kube-demo.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'npm install'
                sh 'node index.js & sleep 2 && curl -f http://localhost:3000'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
                sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin"
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s-deployment.yaml'
            }
        }
    }
}




