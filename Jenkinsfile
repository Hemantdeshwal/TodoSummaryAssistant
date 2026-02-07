pipeline {
  agent any

  tools {
    maven 'M3'
    jdk 'JDK17'        // or JDK21 if that’s what you configured in Jenkins
  }

  environment {
    REGISTRY_URL  = 'hemantdeshwal19'
    IMAGE_NAME    = 'todo-backend'
    REGISTRY_CRED = 'dockerhub-creds'
  }

  options {
    timestamps()
    skipStagesAfterUnstable()
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test Backend') {
      steps {
        dir('Backend/todo-summary-assistant') {
          sh 'mvn clean verify'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def commit = sh(
            script: 'git rev-parse --short HEAD',
            returnStdout: true
          ).trim()

          env.IMAGE_TAG = commit

          dir('Backend/todo-summary-assistant') {
            sh """
              docker build -t ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} .
            """
          }
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: env.REGISTRY_CRED,
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Tag Latest') {
      steps {
        sh """
          docker tag ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY_URL}/${IMAGE_NAME}:latest
          docker push ${REGISTRY_URL}/${IMAGE_NAME}:latest
        """
      }
    }
  }

  post {
    success {
      echo "✅ Build, test, and Docker push completed successfully."
    }
    failure {
      echo "❌ Build failed. Check logs."
    }
  }
}
