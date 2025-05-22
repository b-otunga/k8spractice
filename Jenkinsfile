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

        stage('Install Node.js, Docker & kubectl') {
            steps {
                sh '''
                    set -e

                    echo "[*] Updating system..."
                    apt-get update

                    echo "[*] Installing Node.js 18..."
                    curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                    apt-get update
                    apt-get install -y nodejs

                    echo "[*] Installing Docker dependencies..."
                    apt-get install -y ca-certificates curl gnupg lsb-release

                    echo "[*] Setting up Docker APT keyring..."
                    install -m 0755 -d /etc/apt/keyrings
                    curl -fsSL -o /tmp/docker.gpg https://download.docker.com/linux/ubuntu/gpg

                    if [ ! -s /tmp/docker.gpg ]; then
                        echo "ERROR: Failed to download Docker GPG key"
                        exit 1
                    fi

                    gpg --dearmor --batch -o /etc/apt/keyrings/docker.gpg /tmp/docker.gpg
                    rm /tmp/docker.gpg
                    chmod a+r /etc/apt/keyrings/docker.gpg

                    echo "[*] Adding Docker APT repository..."
                    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

                    echo "[*] Installing Docker CLI and containerd..."
                    apt-get update
                    apt-get install -y docker-ce-cli containerd.io

                    echo "[*] Installing kubectl..."
                    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/arm64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                    echo "[✔] Tools installed successfully."
                '''
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    echo "[*] Installing dependencies..."
                    npm install

                    echo "[*] Starting app and testing endpoint..."
                    node index.js & sleep 2
                    curl -f http://localhost:3000
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                    echo "[*] Building Docker image..."
                    docker build -t $IMAGE_NAME .

                    echo "[*] Logging in to DockerHub..."
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin

                    echo "[*] Pushing Docker image..."
                    docker push $IMAGE_NAME
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "[*] Deploying to Kubernetes..."
                    kubectl apply -f k8s-deployment.yaml
                '''
            }
        }
    }
}
