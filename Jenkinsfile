pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'saadsokkary/jenkins-nginx'
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
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                        sh 'docker logout'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sshagent(['ec2-ssh-key']) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                                    docker pull ${DOCKER_IMAGE}:${IMAGE_TAG}
                                    docker stop ${CONTAINER_NAME} || true
                                    docker rm ${CONTAINER_NAME} || true
                                    docker run -d --name ${CONTAINER_NAME} -p 80:80 ${DOCKER_IMAGE}:${IMAGE_TAG}
                                    docker logout
                                '
                            """
                        }
                    }
                }
            }
        }
    }
}
