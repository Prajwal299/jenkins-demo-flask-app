pipeline {
  agent any
  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/your-username/flask-app.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("flask-app:latest")
        }
      }
    }
    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh """
            echo $PASS | docker login -u $USER --password-stdin
            docker tag flask-app:latest $USER/flask-app:latest
            docker push $USER/flask-app:latest
          """
        }
      }
    }
    stage('Deploy to EC2') {
      steps {
        sshagent(['ec2-ssh-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ec2-user@<EC2-IP> '
              docker pull <dockerhub-user>/flask-app:latest &&
              docker stop flask || true &&
              docker rm flask || true &&
              docker run -d --name flask -p 5000:5000 <dockerhub-user>/flask-app:latest
            '
          '''
        }
      }
    }
  }
}
