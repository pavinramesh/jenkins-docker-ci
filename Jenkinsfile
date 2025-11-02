pipeline {
  agent any
  environment {
    DOCKER_HOST = "tcp://dind:2375"
    IMAGE_NAME  = "hello-ci"
    CONTAINER_NAME = "hello-app"
    HOST_PORT = "8081"
    APP_PORT  = "3000"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Install & Test') {
      steps {
        sh '''
          node -v || true
          npm ci || npm install
          npm test || true
        '''
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
      }
    }
    stage('Run Container') {
      steps {
        sh '''
          docker rm -f ${CONTAINER_NAME} || true
          docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${APP_PORT} ${IMAGE_NAME}:${BUILD_NUMBER}
        '''
      }
    }
  }
  post {
    success {
      echo "App should be live at http://localhost:${HOST_PORT}"
    }
  }
}
