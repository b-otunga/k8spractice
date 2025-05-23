pipeline {
    // CHANGE THIS LINE
    agent { label 'dind-agent' } 

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

        stage('Install Node.js & Tools') {
            steps {
                sh '''
                    # Install Node.js
                    curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                    apt-get update
                    apt-get install -y nodejs

                    # Install kubectl (needed on the agent for 'kubectl apply')
                    # This assumes the agent base image (jenkins/jnlp-agent) is debian/ubuntu-like
                    apt-get install -y ca-certificates curl gnupg lsb-release
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                '''
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
