pipeline {
  agent any

  triggers { pollSCM('H/5 * * * *') }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    GIT_SHA = "${env.GIT_COMMIT ? env.GIT_COMMIT.take(7) : 'local'}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker sanity check') {
      steps {
        sh '''
          docker version
          docker info
        '''
      }
    }

    stage('Build base image (build)') {
      steps {
        sh '''
          docker build \
            -f Dockerfile.build \
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
            --build-arg BASE_IMAGE=loguru-build:${GIT_SHA} \
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
        docker rmi loguru-test:${GIT_SHA} 2>/dev/null || true
        docker rmi loguru-build:${GIT_SHA} 2>/dev/null || true
      '''
    }
  }
}

