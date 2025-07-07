pipeline {
    agent any

    environment {
        EC2_HOST = "ubuntu@13.60.31.154"
        EC2_APP_DIR = "/home/ubuntu/app"
        IMAGE_NAME = "jenkins-flask-app"
        CONTAINER_NAME = "flask"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Transfer Files to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-jenkins-app', // your EC2 SSH credentials
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat """
                        powershell -Command "scp -i %SSH_KEY% -o StrictHostKeyChecking=no -r * ${EC2_HOST}:${EC2_APP_DIR}"
                    """
                }
            }
        }

        stage('Build & Deploy on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-jenkins-app',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat """
                        ssh -i %SSH_KEY% -o StrictHostKeyChecking=no ${EC2_HOST} "cd ${EC2_APP_DIR} && \
                        docker build -t ${IMAGE_NAME} . && \
                        docker stop ${CONTAINER_NAME} || true && \
                        docker rm ${CONTAINER_NAME} || true && \
                        docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE_NAME}"
                    """
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
        success {
            echo '✅ Deployment successful.'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
