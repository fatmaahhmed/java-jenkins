pipeline {
    agent any

    environment {
        DOCKER_HUB = "fatmahmed/java-app" 
    }

    tools {
        maven 'mvn-3.5-4'
        jdk 'Java-18'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fix mvnw Permissions') {
            steps {
                sh 'chmod +x mvnw'
            }
        }

        stage('Dependency Check') {
            steps {
                sh 'mvn dependency-check:check'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive App') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        sh "docker build -t ${DOCKER_HUB}:latest ."
                        sh "docker push ${DOCKER_HUB}:latest"
                    """
                }
            }
        }
    }
}   