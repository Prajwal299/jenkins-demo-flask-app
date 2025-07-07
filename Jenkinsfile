// pipeline {
//   agent any
//   stages {
//     stage('Clone Repo') {
//       steps {
//         script {
//       git branch: 'main', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
//     }
//       }
//     }
//     stage('Build Docker Image') {
//       steps {
//         script {
//           dockerImage = docker.build("jenkins-flask-app:demo1")
//         }
//       }
//     }
//     stage('Push to DockerHub') {
//       steps {
//         withCredentials([usernamePassword(credentialsId: 'dockerhubs-creds', usernameVariable: 'USER', passwordVariable: "PASS")]) {
//           sh """
//             echo $PASS | docker login -u $USER --password-stdin
//             docker tag jenkins-flask-app:demo1 $USER/jenkins-flask-app:demo1
//             docker push $USER/jenkins-flask-app:demo1
//           """
//         }
//       }
//     }
//     stage('Deploy to EC2') {
//       steps {
//         sshagent(['docker-jenkins-app']) {
//           sh '''
//             ssh -o StrictHostKeyChecking=no ec2-user@13.60.31.154  '
//               docker pull prajwalrawate1/jenkins-flask-app:demo1 &&
//               docker stop flask || true &&
//               docker rm flask || true &&
//               docker run -d --name flask -p 5000:5000 prajwalrawate1/jenkins-flask-app:demo1
//             '
//           '''
//         }
//       }
//     }
//   }
// }




pipeline {
  agent any

  environment {
    IMAGE_NAME = "jenkins-flask-app"
    IMAGE_TAG = "demo1"
    FULL_IMAGE = "prajwalrawate1/demo-app-flask:demo1"
  }

  stages {

    stage('Clone Repo') {
      steps {
        script {
          git branch: 'main', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t demo-app-flask:demo1 ."
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhubs-creds', usernameVariable: prajwalrawate1, passwordVariable: Rawate@123455)]) {
          script {
            sh """
              echo Rawate@123455 | docker login -u prajwalrawate1 --password-stdin
              docker tag demo-app-flask:demo1 prajwalrawate1/jenkins-flask-app:demo1
              docker push prajwalrawate1/jenkins-flask-app:demo1
            """
          }
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(['docker-jenkins-app']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ec2-user@13.60.31.154 '
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
