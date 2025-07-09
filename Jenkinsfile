// Jenkinsfile - No Docker Hub / Registryless approach

pipeline {
    agent any

    environment {
        // IP of the server where the application will be deployed
        DEPLOY_SERVER_IP = "ec2-13-60-31-154.eu-north-1.compute.amazonaws.com"
        
        // A simple name for the Docker image we will build locally on the deployment server
        IMAGE_NAME = "local-flask-app"
        
        // Name for the running container
        CONTAINER_NAME = "flask-app-container"
        
        // Your project's repository URL
        REPO_URL = "https://github.com/Prajwal299/jenkins-demo-flask-app.git"
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 instance: ${DEPLOY_SERVER_IP}"
                // Use the SSH Agent plugin with the credential ID for your EC2 private key
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${DEPLOY_SERVER_IP} '
                            
                            # Define a workspace directory on the deployment server
                            APP_DIR="/home/ubuntu/jenkins-demo-flask-app"

                            echo "--- Connected to deployment server. Preparing workspace... ---"

                            # Clone the repo if it doesn't exist, or pull the latest changes if it does
                            if [ ! -d "\$APP_DIR" ]; then
                                echo "Cloning repository..."
                                git clone ${REPO_URL} \$APP_DIR
                            else
                                echo "Repository exists. Pulling latest changes..."
                                cd \$APP_DIR
                                git pull
                            fi

                            # Navigate into the application directory
                            cd \$APP_DIR

                            echo "--- Building Docker image locally on deployment server... ---"
                            # Build the image using the local Dockerfile
                            # We tag it with the Jenkins build number for history and 'latest' for convenience
                            docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .

                            echo "--- Stopping and removing old container... ---"
                            # Stop the running container if it exists (|| true prevents script failure)
                            docker stop ${CONTAINER_NAME} || true

                            # Remove the old container if it exists
                            docker rm ${CONTAINER_NAME} || true

                            echo "--- Starting new container... ---"
                            # Run the new container from the 'latest' image
                            # We map port 80 on the host to port 5000 in the container
                            docker run -d --name ${CONTAINER_NAME} -p 80:5000 ${IMAGE_NAME}:latest
                            
                            echo "--- Deployment successful! ---"

                            # Optional: Clean up old, untagged images to save space
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