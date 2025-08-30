pipeline {
  agent any
  tools { nodejs 'node' }   // Global Tool name = node

  environment {
    APP_IMAGE      = (env.BRANCH_NAME == 'main') ? 'nodemain' : 'nodedev'
    CONTAINER_NAME = APP_IMAGE
    HOST_PORT      = (env.BRANCH_NAME == 'main') ? '3000' : '3001'
    TAG            = 'v1.0'
  }

  stages {
    stage('Checkout SCM') { steps { checkout scm } }
    stage('Build')        { steps { sh 'npm install' } }
    stage('Test')         { steps { sh 'npm test'    } }

    stage('Docker Build') {
      steps { sh "docker build -t ${APP_IMAGE}:${TAG} ." }
    }

    stage('Deploy') {
      steps {
        sh """
          # удаляем только контейнер соответствующего env, если есть
          docker ps -a --filter "name=^/${CONTAINER_NAME}\$" -q | xargs -r docker rm -f
          docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:3000 ${APP_IMAGE}:${TAG}
        """
      }
    }
  }

  post {
    always {
      echo "Branch=${env.BRANCH_NAME} image=${APP_IMAGE}:${TAG} port=${HOST_PORT}"
    }
  }
}
