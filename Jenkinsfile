pipeline {
  agent { label 'docker-host' }

  environment {
    IMAGE_NAME = "openlabfree/mynginx"
    DEPLOY_REPO = "https://github.com/openlabfree/gitops.git"
    DOCKERHUB_CREDS_ID = "dockerhub-creds"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/openlabfree/nginx-app.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            def shortSha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            sh """
              sudo docker login -u $DOCKER_USER -p $DOCKER_PASS
              sudo docker build -t ${IMAGE_NAME}:${shortSha} .
              sudo docker push ${IMAGE_NAME}:${shortSha}
            """
          }
        }
      }
    }

    stage('Update Kustomize Repo') {
      steps {
        withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_TOKEN')]) {
          script {
            def shortSha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            sh """
              rm -rf gitops
              git clone https://openlabfree:${GITHUB_TOKEN}@github.com/openlabfree/gitops.git

              cd gitops/mynginx-kustomize/overlays/dev
              /home/ubuntu/build/kustomize edit set image ${IMAGE_NAME}=${IMAGE_NAME}:${shortSha}

              git config user.name "CI Bot"
              git config user.email "208937492+openlabfree@users.noreply.github.com"
              git add .
              git commit -am "Update image to ${shortSha}"
              git push https://openlabfree:${GITHUB_TOKEN}@github.com/openlabfree/gitops.git main
            """
          }
        }
      }
    }

  }
}
