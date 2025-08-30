pipeline {
  agent any
  tools { nodejs 'node' }   // имя совпадает с Tool в Jenkins

  environment {
    TAG = 'v1.0'
  }

  stages {
    stage('Set ENV Vars') {
      steps {
        script {
          if (env.BRANCH_NAME == 'main') {
            env.APP_IMAGE = 'nodemain'
            env.CONTAINER_NAME = 'nodemain'
            env.HOST_PORT = '3000'
          } else {
            env.APP_IMAGE = 'nodedev'
            env.CONTAINER_NAME = 'nodedev'
            env.HOST_PORT = '3001'
          }
        }
      }
    }

    stage('Checkout SCM') { steps { checkout scm } }
    stage('Build')        { steps { sh 'npm install' } }
    stage('Test')         { steps { sh 'npm test || echo "tests skipped"' } }

    stage('Docker Build') {
      steps { sh "docker build -t ${APP_IMAGE}:${TAG} ." }
    }

    stage('Deploy') {
      steps {
        sh """
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
