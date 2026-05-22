pipeline {
    
    agent any
    environment {
        IMAGE_NAME = "ashishwayachal12/santra"
        IMAGE_TAG  = "${BUILD_ID}"
        APP_PORT   = "8090"
    }

    stages {
        stage("CODE") {
            steps {
                echo "Cloning the code..."
                git url: "https://github.com/Ashishwayachal12/santra.git", branch: "main"
                echo "Code cloning successful"
            }
        }

        stage("GET PUBLIC IP") {
            steps {
                script {
                    env.SERVER_IP = sh(
                        script: "curl -s https://checkip.amazonaws.com",
                        returnStdout: true
                    ).trim()

                    echo "Current Public IP: ${env.SERVER_IP}"
                }
            }
        }

        stage("BUILD") {
            steps {
                echo "Building Docker image..."
                sh "docker build -t santra:${IMAGE_TAG} ."
            }
        }

        stage("DEPLOY") {
            steps {
                echo "Deploying container..."
                sh '''
                    docker compose up -d --build
                    docker ps -a
                '''
            }
        }

        stage("TAG") {
            steps {
                echo "Creating Docker Hub tag..."
                sh "docker tag santra:${IMAGE_TAG} ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage("PUSH") {
            steps {
                script {
                    echo "Pushing image to Docker Hub..."
                    withDockerRegistry(credentialsId: '8d72d5f5-1a1d-4a72-aec3-4d20f34b4238') {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "DEPLOYMENT SUCCESSFUL"
            mail to: 'ashishwayachal14@gmail.com,kabshataiyub@gmail.com',
                 subject: "SUCCESS: Jenkins Build ${BUILD_NUMBER}",
                 body: """
                Build Status : SUCCESS
                Job Name     : ${JOB_NAME}
                Build Number : ${BUILD_NUMBER}
                Application URL:
                http://${SERVER_IP}:${APP_PORT}
              """
        }

        failure {
            echo "DEPLOYMENT FAILED"
            mail to: 'ashishwayachal14@gmail.com,kabshataiyub@gmail.com',
                 subject: "FAILED: Jenkins Build ${BUILD_NUMBER}",
                 body: """
                 Build Status : FAILED
                 Job Name     : ${JOB_NAME}
                 Build Number : ${BUILD_NUMBER}
                 Check Console Output:
                 ${BUILD_URL}
                 Server IP : ${SERVER_IP}
                 Port      : ${APP_PORT}
           """
        }
    }
}
