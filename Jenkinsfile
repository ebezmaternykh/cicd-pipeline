pipeline {
    agent any

    parameters {
      string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker Image Tag')
    }

    environment {
        DOCKER_IMAGE_NAME = "${BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        NODEJS_VERSION = 'NodeJS 7.8.0'
    }

        stage('Test NodeJS Application') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker stop \$(docker ps -q) || true"
                    sh "docker rm \$(docker ps -a -q) || true"
                    sh "docker rmi -f \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG} || true"
                    sh "docker build -t \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG} ."
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    if (BRANCH_NAME == 'main') {
                        sh 'docker run -d --expose 3000 -p 3000:3000 \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG}'
                    } else if (BRANCH_NAME == 'dev') {
                        sh 'docker run -d --expose 3001 -p 3001:3000 \${DOCKER_IMAGE_NAME}:\${IMAGE_TAG}'
                    }
                }
            }
        }

        stage('Scan Docker Image for Vulnerability') {
            steps {
                script {
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM.LOW --no-progress \
                    ${registry}:${env.BUILD_ID }", returnStdout: true).trim()
                    echo "Vurnerability Report:\n${vulnerabilities}"
                }
            }
        }
    }
}
