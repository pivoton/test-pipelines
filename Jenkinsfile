pipeline {
  agent any

  options {
    timestamps()
  }

  parameters {
    choice(
      name: 'TEST_ENV',
      choices: ['(none)', 'prod', 'acc3', 'acc2'],
      description: 'Voeg optioneel "--env <omgeving>" toe aan pytest'
    )

    string(
      name: 'SITE',
      defaultValue: '',
      description: 'Optioneel: --site <waarde> (alleen gebruiken als TEST_ENV = (none))'
    )
  }

  environment {
    IMAGE_NAME     = "myapp:${BUILD_NUMBER}"
    CONTAINER_NAME = "myapp-tests-${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout project repo') {
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

              git -c http.extraHeader="Authorization: Basic $AUTH" \
                  fetch --depth 1 origin develop

              git checkout -f FETCH_HEAD
            '''
          }
        }
      }
    }

    stage('Build Docker image') {
      steps {
        dir('project') {
          sh '''
            set -euxo pipefail
            docker build -t "$IMAGE_NAME" .
          '''
        }
      }
    }

    stage('Run tests') {
      steps {
        dir('project') {
          script {

            def extraArg = ""

            if (params.TEST_ENV != '(none)') {

              if (params.SITE?.trim()) {
                error("Je kunt geen SITE gebruiken als TEST_ENV gekozen is.")
              }

              extraArg = "--env ${params.TEST_ENV}"

            } else if (params.SITE?.trim()) {

              extraArg = "--site ${params.SITE}"

            }

            sh """
              set -euxo pipefail

              docker run --name "$CONTAINER_NAME" \
                "$IMAGE_NAME" \
                pytest -n 6 --browser chromium ${extraArg}
            """
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