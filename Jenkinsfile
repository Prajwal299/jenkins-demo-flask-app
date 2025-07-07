pipeline {
    agent any

    environment {
        EC2_HOST = "ubuntu@ec2-13-60-31-154.eu-north-1.compute.amazonaws.com"
        REMOTE_DIR = "/home/ubuntu/flask-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-pat', 
                    url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker build -t flask-app .'
                    } else {
                        bat 'docker build -t flask-app .'
                    }
                }
            }
        }

        stage('Save Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker save flask-app -o flask-app.tar'
                    } else {
                        bat 'docker save flask-app -o flask-app.tar'
                    }
                }
            }
        }

        stage('Copy Image to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'PEM_PATH')]) {
                    script {
                        if (isUnix()) {
                            sh """
                            scp -o StrictHostKeyChecking=no -i "${PEM_PATH}" flask-app.tar ${EC2_HOST}:${REMOTE_DIR}/
                            """
                        } else {
                            bat """
                            scp -o StrictHostKeyChecking=no -i "${PEM_PATH}" flask-app.tar ${EC2_HOST}:${REMOTE_DIR}/
                            """
                        }
                    }
                }
            }
        }

        stage('Run App on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'PEM_PATH')]) {
                    script {
                        if (isUnix()) {
                            sh """
                            ssh -o StrictHostKeyChecking=no -i "${PEM_PATH}" ${EC2_HOST} << EOF
                            mkdir -p ${REMOTE_DIR}
                            cd ${REMOTE_DIR}
                            docker load < flask-app.tar
                            docker stop flask-app || true
                            docker rm flask-app || true
                            docker run -d --name flask-app -p 5000:5000 flask-app
                            EOF
                            """
                        } else {
                            bat """
                            ssh -o StrictHostKeyChecking=no -i "${PEM_PATH}" ${EC2_HOST} ^
                            "mkdir -p ${REMOTE_DIR} && ^
                            cd ${REMOTE_DIR} && ^
                            docker load < flask-app.tar && ^
                            docker stop flask-app || true && ^
                            docker rm flask-app || true && ^
                            docker run -d --name flask-app -p 5000:5000 flask-app"
                            """
                        }
                    }
                }
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