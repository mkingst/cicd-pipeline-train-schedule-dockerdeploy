pipeline {
    environment {
      IMAGE_NAME = "mkingst14/train-schedule"
      PORT = "8090"
      CONTAINER_NAME = "train-schedule"
    }
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("${IMAGE_NAME}")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy Container to Production') {
            when {
                branch 'master'
            }
            steps { 
                input 'Deploy to Production'
                milestone(1)
                script {
                    def dockerStop = "docker stop ${CONTAINER_NAME} || true && docker rm ${CONTAINER_NAME} || true"
                    def dockerRun = "docker run -d \
                                      --name ${CONTAINER_NAME} \
                                      --publish ${PORT}:8080 \
                                      ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sshagent(['prod_server']) {
                        sh "ssh -o StrictHostKeyChecking=no admin@34.254.159.143 ${dockerStop}"
                        sh "ssh -o StrictHostKeyChecking=no admin@34.254.159.143 ${dockerRun}"
              }
          }
        }
      }
    }
  }
