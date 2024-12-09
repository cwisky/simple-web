pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ghcr.io/cwisky/simple-app:1.0"
        CONTAINER_NAME = "simple-app-container"
        LOG_FILE = "/var/log/simple-app/app.log"
    }
    stages {
        stage('Prepare Logs') {
            steps {
                script {
                    sh "sudo mkdir -p /var/log/simple-app"
                    sh "sudo touch /var/log/simple-app/app.log"
                    sh "sudo chmod -R 775 /var/log/simple-app"
                    sh "sudo chown -R jenkins:jenkins /var/log/simple-app"
                }
            }
        }
        stage('Pull Docker Image') {
            steps {
                script {
                    sh "sudo docker login ghcr.io -u cwisky -p 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"
                    sh "sudo docker pull ${DOCKER_IMAGE}"
                }
            }
        }
        stage('Stop and Remove Existing Container') {
            steps {
                script {
                    // Stop the container only if it is running
                    sh """
                        if [ \$(sudo docker ps -q -f name=${CONTAINER_NAME}) ]; then
                            echo "Stopping running container: ${CONTAINER_NAME}"
                            sudo docker stop ${CONTAINER_NAME}
                        fi
                    """
                    // Remove the container only if it exists
                    sh """
                        if [ \$(sudo docker ps -a -q -f name=${CONTAINER_NAME}) ]; then
                            echo "Removing existing container: ${CONTAINER_NAME}"
                            sudo docker rm ${CONTAINER_NAME}
                        fi
                    """
                }
            }
        }
        stage('Run Container') {
            steps {
                script {
                    sh """
                        sudo docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 8081:8081 \
                        ${DOCKER_IMAGE}
                    """
                    // Append logs from the container to the log file
                    sh """
                        sudo docker logs -f ${CONTAINER_NAME} >> ${LOG_FILE} 2>&1 &
                    """
                }
            }
        }
    }
}
