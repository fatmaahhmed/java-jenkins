pipeline {
  agent any

  environment {
    DOCKER_HUB = 'fatmahmed/java-app'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhub-cred'
  }

  tools {
    maven 'mvn-3.5-4'   // make sure these tool names exist in Jenkins global config
    jdk   'Java-18'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Fix mvnw Permissions') {
      steps {
        sh 'chmod +x mvnw || true'
      }
    }

    stage('Dependency Check') {
      steps {
        sh 'mvn dependency-check:check'
      }
    }

    stage('Build App') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Archive App') {
      steps {
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
      }
    }

    stage('Docker Build & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-cred',
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_PASS'
        )]) {
          sh """
            set -eux
            echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
            test -f Dockerfile || { echo 'Dockerfile not found'; exit 1; }
            docker build -t ${DOCKER_HUB}:latest .
            docker push ${DOCKER_HUB}:latest
          """
        }
      }
    }
  }

  post {
    always {
      echo "Build finished with status: ${currentBuild.currentResult}"
    }
  }
}
