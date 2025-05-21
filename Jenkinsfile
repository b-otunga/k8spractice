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
                    curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
		    apt-get update
                    apt-get install -y nodejs

                    apt-get install -y ca-certificates curl gnupg lsb-release
                    install -m 0755 -d /etc/apt/keyrings
                    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor --batch -o /etc/apt/keyrings/docker.gpg
                    chmod a+r /etc/apt/keyrings/docker.gpg
                    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
                    apt-get update
                    apt-get install -y docker-ce-cli containerd.io

                    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/arm64/kubectl
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                '''
           } }



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




