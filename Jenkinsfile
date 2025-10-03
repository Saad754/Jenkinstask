pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'saadsokkary/custom-nginx'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        EC2_HOST = '54.152.50.94'
        EC2_USER = 'ubuntu'
        CONTAINER_NAME = 'nginx'
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sshagent(['ec2-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                docker pull ${DOCKER_IMAGE}:${IMAGE_TAG}
                                docker stop ${CONTAINER_NAME} || true
                                docker rm ${CONTAINER_NAME} || true
                                docker run -d --name ${CONTAINER_NAME} -p 80:80 ${DOCKER_IMAGE}:${IMAGE_TAG}
                            '
                        """
                    }
                }
            }
        }
    }
}
