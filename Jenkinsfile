// // pipeline {
// //   agent any
// //   stages {
// //     stage('Clone Repo') {
// //       steps {
// //         script {
// //       git branch: 'main', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
// //     }
// //       }
// //     }
// //     stage('Build Docker Image') {
// //       steps {
// //         script {
// //           dockerImage = docker.build("jenkins-flask-app:demo1")
// //         }
// //       }
// //     }
// //     stage('Push to DockerHub') {
// //       steps {
// //         withCredentials([usernamePassword(credentialsId: 'dockerhubs-creds', usernameVariable: 'USER', passwordVariable: "PASS")]) {
// //           sh """
// //             echo $PASS | docker login -u $USER --password-stdin
// //             docker tag jenkins-flask-app:demo1 $USER/jenkins-flask-app:demo1
// //             docker push $USER/jenkins-flask-app:demo1
// //           """
// //         }
// //       }
// //     }
// //     stage('Deploy to EC2') {
// //       steps {
// //         sshagent(['docker-jenkins-app']) {
// //           sh '''
// //             ssh -o StrictHostKeyChecking=no ec2-user@13.60.31.154  '
// //               docker pull prajwalrawate1/jenkins-flask-app:demo1 &&
// //               docker stop flask || true &&
// //               docker rm flask || true &&
// //               docker run -d --name flask -p 5000:5000 prajwalrawate1/jenkins-flask-app:demo1
// //             '
// //           '''
// //         }
// //       }
// //     }
// //   }
// // }



// pipeline {
//   agent any

//   environment {
//     IMAGE_NAME = "jenkins-flask-app"
//     IMAGE_TAG = "demo1"
//     FULL_IMAGE = "prajwalrawate1/${IMAGE_NAME}:${IMAGE_TAG}"
//   }

//   stages {

//     stage('Clone Repo') {
//       steps {
//         script {
//           git branch: 'main', url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
//         }
//       }
//     }

//     stage('Build Docker Image') {
//       steps {
//         sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
//       }
//     }

//     stage('Push to DockerHub') {
//       steps {
//         withCredentials([usernamePassword(credentialsId: 'dockerhubs-creds-1', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
//           sh '''
//             echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin
//             docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKER_HUB_USERNAME/${IMAGE_NAME}:${IMAGE_TAG}
//             docker push $DOCKER_HUB_USERNAME/${IMAGE_NAME}:${IMAGE_TAG}
//           '''
//         }
//       }
//     }

//     stage('Deploy to EC2') {
//       steps {
//         sshagent(['docker-jenkins-app']) {
//           sh '''
//             ssh -o StrictHostKeyChecking=no ec2-user@13.60.31.154 '
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
                    credentialsId: 'docker-jenkins-app', // SSH key stored in Jenkins
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    script {
                        // Fix Windows key permissions
                        bat """
                            icacls "%SSH_KEY%" /inheritance:r
                            icacls "%SSH_KEY%" /grant:r "%USERNAME%":(R)
                            icacls "%SSH_KEY%" /grant:r "SYSTEM":(R)
                        """

                        // Transfer code to EC2 instance
                        bat """
                            powershell -Command "scp -i %SSH_KEY% -o StrictHostKeyChecking=no -r * %EC2_HOST%:%EC2_APP_DIR%"
                        """
                    }
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
                    script {
                        bat """
                            ssh -i %SSH_KEY% -o StrictHostKeyChecking=no %EC2_HOST% ^
                                "cd %EC2_APP_DIR% && ^
                                 docker build -t %IMAGE_NAME% . && ^
                                 (docker stop %CONTAINER_NAME% || echo 'Not running') && ^
                                 (docker rm %CONTAINER_NAME% || echo 'Not present') && ^
                                 docker run -d --name %CONTAINER_NAME% -p 5000:5000 %IMAGE_NAME%"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
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
