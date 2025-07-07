pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '13.60.31.154'
        EC2_APP_DIR = '/home/ubuntu/app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'docker-jenkins-app', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git', branch: 'main'
            }
        }

        stage('Transfer Files to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-jenkins-app',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    // Fix key permissions on Windows
                    bat """
                        powershell -Command "
                            \$acl = Get-Acl '%SSH_KEY%';
                            \$acl.SetAccessRuleProtection(\$true, \$false);
                            \$acl.Access | Where-Object { \$_.IdentityReference -ne 'NT AUTHORITY\\\\SYSTEM' -and \$_.IdentityReference -ne \$env:USERNAME } | ForEach-Object { \$acl.RemoveAccessRule(\$_) };
                            Set-Acl '%SSH_KEY%' \$acl
                        "
                    """

                    // Transfer files to EC2
                    bat """
                        powershell -Command "scp -i %SSH_KEY% -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:${EC2_APP_DIR}"
                    """
                }
            }
        }

        stage('Build & Run Docker on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-jenkins-app',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat """
                        powershell -Command "ssh -i %SSH_KEY% -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} \\
                        'cd ${EC2_APP_DIR} && docker build -t flask-app . && docker stop flask-app || true && docker rm flask-app || true && docker run -d -p 5000:5000 --name flask-app flask-app'"
                    """
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Deployment failed.'
        }
        success {
            echo '✅ Deployment succeeded!'
        }
        always {
            cleanWs()
        }
    }
}
