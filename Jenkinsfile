pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE_NAME     = "myapp:${BUILD_NUMBER}"
    CONTAINER_NAME = "myapp-tests-${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout project repo (manual)') {
      steps {
        dir('project') {
          deleteDir()
          withCredentials([usernamePassword(
            credentialsId: 'github-classic-pat',
            usernameVariable: 'GIT_USER',
            passwordVariable: 'GIT_PAT'
          )]) {
            sh '''
              set -euo pipefail
              git init
              git remote add origin https://github.com/pivoton/oplever-controle.git
              git config --local credential.helper ""

              AUTH="$(printf "%s:%s" "$GIT_USER" "$GIT_PAT" | base64 | tr -d '\\n')"
              git -c http.extraHeader="Authorization: Basic $AUTH" fetch --depth 1 origin develop
              git checkout -f FETCH_HEAD
            '''
          }
        }
      }
    }

    stage('Build') {
      steps {
        dir('project') {
          sh '''
            set -euxo pipefail
            docker build -t "$IMAGE_NAME" .
          '''
        }
      }
    }

    stage('Test') {
      steps {
        dir('project') {
          withCredentials([file(credentialsId: 'test-env-file', variable: 'ENV_FILE')]) {
            sh '''
              set -euxo pipefail

              # draait tests met .env uit Jenkins secret file
              docker run --name "$CONTAINER_NAME" \
                --env-file "$ENV_FILE" \
                "$IMAGE_NAME"
            '''
          }
        }
      }
    }
  }

  post {
    always {
      sh '''
        set +e
        docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
      '''
    }
  }
}