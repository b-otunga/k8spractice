pipeline {
    agent any

    environment {
        IMAGE_NAME = "botunga/node-kube-demo"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {
        stage('Clone') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/b-otunga/k8spractice.git'
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
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"

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
