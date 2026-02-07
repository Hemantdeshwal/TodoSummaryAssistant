pipeline {
  agent any
  
tools {  // ← ADD THIS - uses your configured Maven
    maven 'M3'  
  }

  environment {
    REGISTRY_URL = 'hemantdeshwal19'         
    IMAGE_NAME   = 'todo-backend'
    REGISTRY_CRED = 'dockerhub-creds'         
  }

  options {
    timestamps()
    skipStagesAfterUnstable()
  }


  stages {
    stage('Build & Test Backend') {
      steps {
        dir('Backend/todo-summary-assistant') {
          sh 'mvn clean verify'  // ← CHANGED: plain mvn, no ./mvnw
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

    stage('Tag Latest (optional)') {
      steps {
        script {
          sh """
            docker tag ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY_URL}/${IMAGE_NAME}:latest
            docker push ${REGISTRY_URL}/${IMAGE_NAME}:latest
          """
        }
      }
    }
  }

  post {
    failure {
      echo "Build failed. Check logs."
    }
  }
}
