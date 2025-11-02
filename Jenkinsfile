pipeline {
  agent any
  environment {
    DOCKER_HOST    = "tcp://dind:2375"   // talk to the Docker-in-Docker helper
    IMAGE_NAME     = "hello-ci"
    CONTAINER_NAME = "hello-app"
    HOST_PORT      = "8081"
    APP_PORT       = "3000"
  }

  stages {
    // Declarative pipelines already do a checkout automatically.
    // (You can keep your manual Checkout stage if you want, but it's redundant.)

    stage('Install & Test') {
      // Run this stage inside a Node.js container
      agent {
        docker {
          image 'node:20-alpine'
          // Cache npm to speed up repeated builds (optional)
          args '-v /root/.npm-cache:/root/.npm'
        }
      }
      steps {
        sh '''
          node -v
          npm -v
          npm ci || npm install
          npm test || true
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker version || true'
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
      echo "App should be live at: http://localhost:${HOST_PORT}"
    }
  }
}
