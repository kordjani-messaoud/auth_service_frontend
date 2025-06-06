pipeline {
    agent any

    environment {
        CONTAINER_REG = "cicd-worker-01.icosnet.local"
        VERSION = "1.1.0"
        IMAGE_NAME = "my-app-frontend"
        IMAGE_TAG = "latest"
        PROJECT =  "sso"
        IMAGE_FULL_NAME = "${CONTAINER_REG}/${PROJECT}/${IMAGE_NAME}:${VERSION}"
        DOCKERFILE_PATH = "./Dockerfile"
        CONTEXT = "."

        PATH = "/bin:/usr/bin:/usr/local/bin"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                        dockerImage = docker.build("${IMAGE_FULL_NAME}", "-f ${DOCKERFILE_PATH} ${CONTEXT}")
                }
            }
        }

        stage('Push to Local Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Harbor', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                sh """
                    echo "$HARBOR_PASS" | docker login ${CONTAINER_REG} -u "$HARBOR_USER" --password-stdin 
                    docker push ${IMAGE_FULL_NAME}
                    docker logout ${CONTAINER_REG}
                """
                }
            }
        }
    }
    post {
        success {
            echo "Docker image successfully built and pushed to container-reg.icosnet.local"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
