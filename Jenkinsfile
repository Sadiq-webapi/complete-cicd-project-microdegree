pipeline {
    agent any
    environment {
        // Define these once so they are the same everywhere
        DOCKER_IMAGE = "mohamedsadiq9741/twitter-app" 
        IMAGE_TAG = "${env.GIT_COMMIT}"
    }
    stages {
        stage('Build & Tag Docker Image') {
            steps {
                bat "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }
       stage('Docker Image Scan') {
    steps {
       bat '''
docker run --rm ^
-v //./pipe/docker_engine://./pipe/docker_engine ^
aquasec/trivy image mohamedsadiq9741/twitter-app:%GIT_COMMIT%
'''
    }
}
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat "docker login -u %USER% -p %PASS%"
                    // Use the same variable here too!
                    bat "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
    }
}
