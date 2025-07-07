pipeline {
    agent any

    environment {
        PEM_PATH = "/c/Users/PRAJWAL RAWATE/.ssh/docker-jenkins-app.pem"
        EC2_HOST = "ubuntu@ec2-13-60-31-154.eu-north-1.compute.amazonaws.com"
        REMOTE_DIR = "/home/ubuntu/flask-app"
        BASH = "\"C:\\Program Files\\Git\\bin\\bash.exe\" -c"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t flask-app .'
            }
        }

        stage('Save Docker Image') {
            steps {
                bat 'docker save flask-app -o flask-app.tar'
            }
        }

        stage('Copy Image to EC2') {
            steps {
                bat """
                ${BASH} 'scp -o StrictHostKeyChecking=no -i "${PEM_PATH}" flask-app.tar ${EC2_HOST}:${REMOTE_DIR}/'
                """
            }
        }

        stage('Run App on EC2') {
            steps {
                bat """
                ${BASH} 'ssh -o StrictHostKeyChecking=no -i "${PEM_PATH}" ${EC2_HOST} << EOF
                mkdir -p ${REMOTE_DIR}
                cd ${REMOTE_DIR}
                docker load < flask-app.tar
                docker stop flask-app || true
                docker rm flask-app || true
                docker run -d --name flask-app -p 5000:5000 flask-app
                EOF'
                """
            }
        }
    }

    post {
        success {
            echo '✅ App Deployed to EC2 Successfully!'
        }
        failure {
            echo '❌ Deployment Failed. Check Logs.'
        }
    }
}
