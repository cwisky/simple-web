pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "cwisky/spring-web:1.0"
        GITHUB_REPO = "https://github.com/cwisky/simple-web.git"
        JAR_FILE = "spring-web.jar"
    }

    stages {
        
        stage('Cleanup Workspace') {
            steps {
                //cleanWs()
                echo 'Cleaned up'
            }
        }

        stage('Git Clone') {
            steps {
                echo 'Cloning Git repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        credentialsId: 'GitHub_ID_PWD',       // Jenkins에서 github의 id, pwd를 등록하고 아이디를 'GitHub_ID_PWD' 으로 지정해야 함
                        url: GITHUB_REPO
                    ]]
                ])
                // JAR 파일에 실행 권한 추가
                sh 'chmod +x ./${JAR_FILE}'
            }
        }

        stage('Prepare JAR File') {
            steps {
                script {
                    echo "Preparing JAR file..."
                    //dir('app') {
                        sh """
                        if [ -f ${JAR_FILE} ]; then
                            echo "JAR file ${JAR_FILE} found."
                        else
                            echo "JAR file not found. Ensure the project is built correctly."
                            exit 1
                        fi
                        """
                   // }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..." 
                    writeFile file: 'Dockerfile', text: """
                    FROM openjdk:21-slim
                    COPY ${JAR_FILE} /spring-web.jar
                    CMD ["java", "-jar", "/spring-web.jar"]
                    """
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        /*
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Docker_Hub_id_pwd', usernameVariable: 'DOCKER_HUB_CREDENTIALS_USR', passwordVariable: 'DOCKER_HUB_CREDENTIALS_PSW')]) {
                        echo "Pushing Docker image to Docker Hub..."
                        sh """
                        echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }*/
        
        stage('Run Docker Container') {
            steps {
                script {
                    echo "Running Docker container..."
                    sh """
                    docker ps -q --filter 'ancestor=${DOCKER_IMAGE}' | xargs --no-run-if-empty docker stop
                    docker run -d -p 80:80 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
    /*
    post {
        always {
            echo "Cleaning up workspace..."
            //cleanWs()
        }
    }*/
}
