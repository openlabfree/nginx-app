pipeline {
  
  agent { label 'docker' }

  environment {
    IMAGE_NAME = "openlabfree/mynginx"
    DEPLOY_REPO = "https://github.com/openlabfree/gitops.git"
    DOCKERHUB_CREDS_ID = "dockerhub-creds"
    GIT_CREDENTIALS_ID = "github-creds"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/yourusername/nginx-site-code.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            def shortSha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            sh """
              docker login -u $DOCKER_USER -p $DOCKER_PASS
              docker build -t ${IMAGE_NAME}:${shortSha} .
              docker push ${IMAGE_NAME}:${shortSha}
            """
          }
        }
      }
    }

    stage('Install kustomize') {
      steps {
        sh 'curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash'
      }
    }

    stage('Update Kustomize Repo') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
          script {
            def shortSha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            sh """
              git clone https://\${GIT_USER}:\${GIT_PASS}@github.com/openlabfree/gitops.git
              cd gitops/mynginx-kustomize
              kustomize edit set image ${IMAGE_NAME}=${IMAGE_NAME}:${shortSha}
              git config user.name "CI Bot"
              git config user.email "208937492+openlabfree@users.noreply.github.com"
              git add .
              git commit -am "Update image to ${shortSha}"
              git push origin main
            """
          }
        }
      }
    }
  }
}
