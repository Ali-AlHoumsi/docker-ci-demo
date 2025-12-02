pipeline {
  agent any

  environment {
    IMAGE_NAME = "Ali Houmsi/docker-ci-demo"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        script {
          def shortSha = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          env.IMAGE_TAG = "${env.IMAGE_NAME}:${shortSha}"
          sh "docker build -t ${env.IMAGE_TAG} ."
          sh "docker tag ${env.IMAGE_TAG} ${env.IMAGE_NAME}:latest"
        }
      }
    }

    stage('Login to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
        }
      }
    }

    stage('Push Image') {
      steps {
        sh "docker push ${env.IMAGE_TAG}"
        sh "docker push ${env.IMAGE_NAME}:latest"
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
    success {
      echo "Image pushed: ${env.IMAGE_TAG}"
    }
    failure {
      echo "Build failed"
    }
  }
}
