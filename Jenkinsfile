// Jenkinsfile (bez deploy)
pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  triggers {
    // Poll SCM co 5 minut
    pollSCM('H/5 * * * *')
  }

  environment {
    // krótki SHA do tagów obrazów
    GIT_SHA = "${env.GIT_COMMIT ? env.GIT_COMMIT.take(7) : 'local'}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'git rev-parse --short HEAD'
      }
    }

    stage('Build base image (build)') {
      steps {
        sh '''
          docker build \
            -f Dockerfile \
            -t loguru-build:${GIT_SHA} \
            -t loguru-build:latest \
            .
        '''
      }
    }

    stage('Build test image & Run tests') {
      steps {
        sh '''
          docker build \
            -f Dockerfile.test \
            -t loguru-test:${GIT_SHA} \
            -t loguru-test:latest \
            .

          docker run --rm loguru-test:${GIT_SHA}
        '''
      }
    }
  }

  post {
    always {
      sh '''
        docker image prune -f || true
      '''
    }
    success {
      echo "OK: build + test zakończone."
    }
    failure {
      echo "FAIL: build lub testy nie przeszły."
    }
  }
}
