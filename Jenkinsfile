pipeline {
  agent any

  triggers { pollSCM('H/5 * * * *') }

  options {
    timestamps()
    disableConcurrentBuilds()
    timeout(time: 5, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '30'))
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
        sh 'docker version'
        sh 'docker info'
      }
    }

    stage('Build build-image') {
      steps {
        sh '''
          docker build -f Dockerfile.build \
            -t loguru-build:${GIT_SHA} \
            -t loguru-build:latest \
            .
        '''
      }
    }

    stage('Build test-image') {
      steps {
        sh '''
          docker build -f Dockerfile.test \
            --build-arg BASE_IMAGE=loguru-build:${GIT_SHA} \
            -t loguru-test:${GIT_SHA} \
            -t loguru-test:latest \
            .
        '''
      }
    }

   stage('Run tests') {
  steps {
    sh '''
      mkdir -p artifacts
      
      docker run --rm loguru-test:${GIT_SHA} | tee artifacts/test.log
    '''
  }
}

post {
  always {
    archiveArtifacts artifacts: 'artifacts/*.log', fingerprint: true
    sh '''
      docker rmi loguru-test:${GIT_SHA} 2>/dev/null || true
      docker rmi loguru-build:${GIT_SHA} 2>/dev/null || true
    '''
  }
}

