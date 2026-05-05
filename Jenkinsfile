pipeline {
    agent any
    environment {
        // Centralize your image name here
        DOCKER_REPO = "mohamedsadiq9741/twitter-app"
        IMAGE_TAG   = "${env.GIT_COMMIT}"
    }
    stages {
        stage('Build') {
            steps {
                bat "docker build -t ${DOCKER_REPO}:${IMAGE_TAG} ."
            }
        }
        stage('Docker Image Scan') {
            steps {
                // If this keeps hanging, add --reset to clear cache or check network
                bat "docker run --rm aquasec/trivy image ${DOCKER_REPO}:${IMAGE_TAG}"
            }
        }
        stage('Push to Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat "docker login -u %USER% -p %PASS%"
                    bat "docker push ${DOCKER_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }
}
