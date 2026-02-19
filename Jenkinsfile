pipeline {
  agent any

  triggers {
    pollSCM('H/5 * * * *')
  }

  options {
    timestamps()
    disableConcurrentBuilds()

  }

  environment {
    GIT_SHA = "${env.GIT_COMMIT ? env.GIT_COMMIT.take(7) : 'local'}"

    DOCKERHUB_REPO = 'matgraa/loguru'
    DOCKERHUB_CREDS_ID = '92886ae8-76dc-4576-9b84-fe958929870a'
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
          ##set -euo pipefail
          mkdir -p artifacts

          docker run --rm loguru-test:${GIT_SHA} \
            | tee artifacts/pytest-output.log
        '''
      }
    }

    stage('Tag & Push to Docker Hub') {
      

      steps {
        sh '''
          docker tag loguru-build:${GIT_SHA} ${DOCKERHUB_REPO}:${GIT_SHA}
          docker tag loguru-build:${GIT_SHA} ${DOCKERHUB_REPO}:latest
          docker tag loguru-build:${GIT_SHA} ${DOCKERHUB_REPO}:build-${BUILD_NUMBER}
        '''

        withCredentials([usernamePassword(
          credentialsId: "${DOCKERHUB_CREDS_ID}",
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_PASS'
        )]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push ${DOCKERHUB_REPO}:${GIT_SHA}
            docker push ${DOCKERHUB_REPO}:build-${BUILD_NUMBER}
            docker push ${DOCKERHUB_REPO}:latest
            docker logout
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'artifacts/*.log', fingerprint: true

      sh '''
        docker rmi loguru-test:${GIT_SHA} 2>/dev/null || true
        docker rmi loguru-build:${GIT_SHA} 2>/dev/null || true
        docker rmi ${DOCKERHUB_REPO}:${GIT_SHA} 2>/dev/null || true
        docker rmi ${DOCKERHUB_REPO}:build-${BUILD_NUMBER} 2>/dev/null || true
        docker rmi ${DOCKERHUB_REPO}:latest 2>/dev/null || true
      '''
    }
  }
}
