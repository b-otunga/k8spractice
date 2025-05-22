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

        stage('Install Node.js') {
            steps {
                sh '''
                    # Install sudo if it's not already present on the agent
                    apt-get update
                    apt-get install -y sudo

                    # Install Node.js
                    curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                    apt-get update
                    apt-get install -y nodejs

                    # Install Docker CLI and Kubectl prerequisites
                    apt-get install -y ca-certificates curl gnupg lsb-release

                    # Create directory for APT keyrings
                    install -m 0755 -d /etc/apt/keyrings

                    # Download and dearmor Docker GPG key using sudo and batch mode
                    # Use 'sudo' if the Jenkins user needs elevated privileges to write to /etc/apt/keyrings
                    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor --batch -o /etc/apt/keyrings/docker.gpg
                    
                    # Set appropriate permissions for the GPG key file
                    chmod a+r /etc/apt/keyrings/docker.gpg

                    # Add Docker APT repository
                    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
                    
                    # Update package lists after adding Docker repo
                    apt-get update
                    
                    # Install Docker CE CLI and containerd.io
                    apt-get install -y docker-ce-cli containerd.io

                    # Download and install kubectl for arm64
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/arm64/kubectl
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
                // Login to Docker Hub using credentials from Jenkins's credential store
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
