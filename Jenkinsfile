pipeline {
  agent any

  environment {
    IMAGE = "soultan874/demo-app"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy4-jenkins-kelompok5"
  }

  stages {
    stage('Checkout Source Code') {
      steps {
        git url: 'https://github.com/soul222/casestudy-jenkins.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building image ${IMAGE}:${TAG}..."
          def builtImage = docker.build("${IMAGE}:${TAG}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "docker-hub",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          script {
            echo "📦 Pushing image to DockerHub..."
            sh """
              echo "$PASS" | docker login -u "$USER" --password-stdin
              docker push ${IMAGE}:${TAG}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          script {
            echo "🚀 Deploying to Kubernetes via Helm..."
            sh '''
              export KUBECONFIG=$KUBE_FILE
              
              # Install helm if not exists in Jenkins container
              if ! command -v helm &> /dev/null; then
                curl https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz | tar xz
                mv linux-amd64/helm /usr/local/bin/
              fi
              
              # Deploy with Helm
              helm upgrade --install $HELM_RELEASE ./helm \
                --set image.repository=$IMAGE \
                --set image.tag=$TAG \
                --namespace $NAMESPACE --create-namespace \
                --wait
            '''
          }
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          script {
            sh '''
              export KUBECONFIG=$KUBE_FILE
              echo "🔍 Checking deployment status..."
              kubectl get pods -n $NAMESPACE
              kubectl get services -n $NAMESPACE
            '''
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dideploy ke Kubernetes"
    }
    failure {
      echo "❌ Pipeline Gagal: Cek log untuk mengetahui error"
    }
  }
}
