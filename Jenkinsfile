pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'yourdockerhubusername/yourimagename'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Clone') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/b-otunga/k8spractice.git'
            }
        }

        stage('Install Node.js & kubectl') {
            steps {
                sh '''
                    set -e

                    echo "[*] Updating system..."
                    apt-get update

                    echo "[*] Installing Node.js 18..."
                    curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                    apt-get update
                    apt-get install -y nodejs

                    echo "[*] Installing kubectl..."
                    curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                '''
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    echo "[*] Installing dependencies..."
                    npm install

                    echo "[*] Running tests..."
                    npm test || true
                '''
            }
        }

        stage('Docker Build & Push') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
            }
            steps {
                sh '''
                    echo "[*] Logging in to Docker Hub..."
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin

                    echo "[*] Building Docker image..."
                    docker build -t $DOCKER_IMAGE:$DOCKER_TAG .

                    echo "[*] Pushing Docker image..."
                    docker push $DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "[*] Deploying to Kubernetes..."
                    kubectl apply -f k8s/
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check logs above.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
    }
}
