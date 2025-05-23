pipeline {
  agent {
    kubernetes {
      label 'docker-agent'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:20.10.7
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }

  environment {
    IMAGE_NAME = "botunga/node-kube-demo"
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
  }

  stages {
    stage('Clone') {
      steps {
        container('docker') {
          git credentialsId: 'github-creds', url: 'https://github.com/b-otunga/k8spractice.git'
        }
      }
    }

    stage('Install Node.js & Tools') {
      steps {
        container('docker') {
          sh '''
            # Install Node.js
            curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
            apt-get update
            apt-get install -y nodejs

            # Install kubectl
            apt-get install -y ca-certificates curl gnupg lsb-release
            curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
            chmod +x kubectl
            mv kubectl /usr/local/bin/
          '''
        }
      }
    }

    stage('Build & Test') {
      steps {
        container('docker') {
          sh 'npm install'
          sh 'node index.js & sleep 2 && curl -f http://localhost:3000'
        }
      }
    }

    stage('Docker Build & Push') {
      steps {
        container('docker') {
          sh 'docker build -t $IMAGE_NAME .'
          sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
          sh 'docker push $IMAGE_NAME'
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('docker') {
          sh 'kubectl apply -f k8s-deployment.yaml'
        }
      }
    }
  }
}
