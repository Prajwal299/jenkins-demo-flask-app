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
        IMAGE_NAME = "jenkins-flask-app"
        IMAGE_TAG = "demo1"
        DOCKER_HUB_REPO = "prajwalrawate1/jenkins-flask-app"
        EC2_INSTANCE = "ec2-user@13.60.31.154"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/Prajwal299/jenkins-demo-flask-app.git'
                        ]]
                    ])
                    echo "Repository cloned successfully"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                        echo "Docker image built successfully"
                    } catch (Exception e) {
                        error("Docker build failed: ${e.getMessage()}")
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhubs-creds-1',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    script {
                        try {
                            bat """
                                echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                                docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                                docker logout
                            """
                            echo "Image pushed to Docker Hub successfully"
                        } catch (Exception e) {
                            bat "docker logout"
                            error("Failed to push image to Docker Hub: ${e.getMessage()}")
                        }
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['docker-jenkins-app']) {
                    script {
                        try {
                            bat """
                                ssh -o StrictHostKeyChecking=no ${EC2_INSTANCE} "
                                    docker pull ${DOCKER_HUB_REPO}:${IMAGE_TAG} && 
                                    (docker stop flask || true) && 
                                    (docker rm flask || true) && 
                                    docker run -d --name flask -p 5000:5000 ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                                "
                            """
                            echo "Application deployed successfully to EC2"
                        } catch (Exception e) {
                            error("Deployment failed: ${e.getMessage()}")
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
            cleanWs()
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}