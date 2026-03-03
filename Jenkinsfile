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
                    sh 'docker build -t "$IMAGE_NAME" .'
                }
            }
        }

        stage('Test') {
            steps {
                dir('project') {
                    sh '''
            docker run -d --name "$CONTAINER_NAME" "$IMAGE_NAME" tail -f /dev/null
            docker exec "$CONTAINER_NAME" sh -lc "npm ci && npm test"
          '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f "$CONTAINER_NAME" 2>/dev/null || true'
        }
    }
}
