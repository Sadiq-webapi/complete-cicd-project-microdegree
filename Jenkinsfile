pipeline {
    agent any

    environment {
        DOCKER_REPO = "mohamedsadiq9741/twitter-app"
        IMAGE_TAG   = "${env.GIT_COMMIT}"
        TRIVY_PATH  = "C:\\ProgramData\\chocolatey\\bin\\trivy.exe"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat "docker build -t %DOCKER_REPO%:%IMAGE_TAG% ."
            }
        }

   stage('Docker Login') {
    script {
        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
            // Jenkins handles the login/logout automatically here
        }
    }
}

        stage('Push Image') {
            steps {
                bat "docker push %DOCKER_REPO%:%IMAGE_TAG%"
                bat "docker tag %DOCKER_REPO%:%IMAGE_TAG% %DOCKER_REPO%:latest"
                bat "docker push %DOCKER_REPO%:latest"
            }
        }

        stage('Deploy') {
            steps {
                // Stop old container if exists
                bat """
                docker stop twitter-container || exit 0
                docker rm twitter-container || exit 0
                """

                // Run new container
                bat """
                docker run -d -p 8081:8080 --name twitter-container %DOCKER_REPO%:%IMAGE_TAG%
                """
            }
        }
    }

    post {

        always {
            echo "Cleaning up unused Docker images..."
            bat "docker system prune -f"
        }

        success {
            echo "Pipeline completed successfully 🚀"
        }

        failure {
            echo "Pipeline failed ❌"
        }
    }
}
