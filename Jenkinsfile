pipeline {
    environment {
      IMAGE_NAME = "mkingst14/train-schedule"
      PORT = "8090"
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
                    app = docker.build("<DOCKER_HUB_USERNAME>/train-schedule")
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
        stage('Deploy Container on Server') {
              steps {
                  sh "docker stop ${IMAGE_NAME} || true && docker rm ${IMAGE_NAME} || true"
                  sh "docker run -d \
                      --name ${IMAGE_NAME} \
                      --publish ${PORT}:8080 \
                      ${IMAGE_NAME}:${env.BUILD_NUMBER}"
              }
          }
      }
    }
