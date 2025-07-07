pipeline {
  agent any
  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("jenkins-flask-app:demo1")
        }
      }
    }
    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhubs-creds', usernameVariable: 'prajwal', passwordVariable: 'Rawate')]) {
          sh """
            echo $PASS | docker login -u $USER --password-stdin
            docker tag jenkins-flask-app:demo1 $USER/jenkins-flask-app:demo1
            docker push $USER/jenkins-flask-app:demo1
          """
        }
      }
    }
    stage('Deploy to EC2') {
      steps {
        sshagent(['docker-jenkins-app']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ec2-user@13.60.31.154  '
              docker pull prajwalrawate1/jenkins-flask-app:demo1 &&
              docker stop flask || true &&
              docker rm flask || true &&
              docker run -d --name flask -p 5000:5000 prajwalrawate1/jenkins-flask-app:demo1
            '
          '''
        }
      }
    }
  }
}
