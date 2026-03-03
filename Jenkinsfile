pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE_NAME     = "myapp:${BUILD_NUMBER}"
    CONTAINER_NAME = "myapp-tests-${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout project repo') {
      steps {
        dir('project') {
          checkout([$class: 'GitSCM',
            branches: [[name: '*/develop']],
            userRemoteConfigs: [[
              url: 'https://github.com/pivoton/oplever-controle.git',
              credentialsId: 'github-https-pat'
            ]]
          ])
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