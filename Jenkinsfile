pipeline {
    agent any

    parameters {
      string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker Image Tag')
      string(name: 'DOCKER_REPO', defaultValue: 'ebezmaternykh', description: 'Docker Hub Repository')
    }

    environment {
        DOCKER_IMAGE_NAME = "${BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        NODEJS_VERSION = 'NodeJS 7.8.0'
    }

    stages {

        stage('Setup NodeJS') {
            tools {
                nodejs NODEJS_VERSION
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Test NodeJS Application') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                  if (BRANCH_NAME == 'main') {
                        sh "docker stop \$(docker ps -q --filter name=nodemain) || true"
                        sh "docker rm \$(docker ps -a -q --filter name=nodemain) || true"
                  } 
                  else if (BRANCH_NAME == 'dev') {
                        sh "docker stop \$(docker ps -q --filter name=nodedev) || true"
                        sh "docker rm \$(docker ps -a -q --filter name=nodedev) || true"
                  }
                  sh "docker rmi -f \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG} || true"
                  sh "docker build -t \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG} ."
                }
            }
        }

        stage('Push to Docker Hub') {
          steps {
            script {
                docker.withRegistry('https://registry.hub.docker.com', 'ebezmaternykh_dockerhub') {
                  sh "docker tag \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG} \${DOCKER_REPO}/\${DOCKER_IMAGE_NAME}:\${IMAGE_TAG}"
                  def dockerImage = docker.image("\${DOCKER_REPO}/\${DOCKER_IMAGE_NAME}:\${IMAGE_TAG}")
                  dockerImage.push()
                }
            }
          }
        }

    }

    post {
      success {
        script {
          if (BRANCH_NAME == 'main') {
                build(job: 'Deploy_to_main')
          } else if (BRANCH_NAME == 'dev') {
              build(job: 'Deploy_to_dev')
          }
        }
      }
    }

}
