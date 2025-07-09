// Jenkinsfile - Corrected Version

pipeline {
    agent any

    environment {
        DEPLOY_SERVER_IP = "ec2-13-60-31-154.eu-north-1.compute.amazonaws.com"
        IMAGE_NAME = "local-flask-app"
        CONTAINER_NAME = "flask-app-container"
        REPO_URL = "https://github.com/Prajwal299/jenkins-demo-flask-app.git"
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 instance: ${DEPLOY_SERVER_IP}"
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${DEPLOY_SERVER_IP} '
                            
                            APP_DIR="/home/ubuntu/jenkins-demo-flask-app"

                            echo "--- Connected to deployment server. Preparing workspace... ---"

                            if [ ! -d "\$APP_DIR" ]; then
                                echo "Cloning repository..."
                                git clone ${REPO_URL} \$APP_DIR
                            else
                                echo "Repository exists. Pulling latest changes..."
                                cd \$APP_DIR
                                git pull
                            fi

                            # ***************************************************************
                            # *** THE FIX IS HERE: Navigate into the application directory ***
                            # ***************************************************************
                            cd \$APP_DIR

                            echo "--- Building Docker image locally on deployment server... ---"
                            docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .

                            echo "--- Stopping and removing old container... ---"
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true

                            echo "--- Starting new container... ---"
                            docker run -d --name ${CONTAINER_NAME} -p 80:5000 ${IMAGE_NAME}:latest
                            
                            echo "--- Deployment successful! ---"
                            docker image prune -f
                        '
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}