pipeline {
    agent any

    environment {
        DOCKER_REPO = "mohamedsadiq9741/twitter-app"
        IMAGE_TAG   = "${env.GIT_COMMIT}"
    }

    stages {

        stage('Build') {
            steps {
                bat "docker build -t %DOCKER_REPO%:%IMAGE_TAG% ."
            }
        }

  stage('Docker Image Scan') {
    steps {
        bat "\"C:\\ProgramData\\chocolatey\\bin\\trivy.exe\" image --timeout 10m %DOCKER_REPO%:%IMAGE_TAG%"
    }
}

        stage('Push to Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat "echo %PASS% | docker login -u %USER% --password-stdin"
                    bat "docker push %DOCKER_REPO%:%IMAGE_TAG%"
                }
            }
        }
    }
}
