pipeline {
    agent any

    environment {
        // IMAGE_NAME is used for tagging and scanning
        IMAGE_NAME = "manojkrishnappa/fullstack:${GIT_COMMIT}"
        AWS_REGION = "ap-south-2"
        CLUSTER_NAME = "my-complete-eks"
        NAMESPACE = "sadiq"
        SONAR_URL = "http://localhost:9000"
        SONAR_TOKEN = "squ_18a41c2ce98900a9ff4d7cd40e28c5ba8b824139"
        DOCKER_HUB_USER = "manojkrishnappa" // Added for cleaner login
    }

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ManojKRISHNAPPA/complete-cicd-project-microdegree.git'
            }
        }

        stage('Compile') {
            steps {
                bat "mvn compile"
            }
        }

        stage('Build & Unit Test') {
            steps {
                bat "mvn package"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    bat "mvn sonar:sonar -Dsonar.projectKey=devops -Dsonar.host.url=${SONAR_URL} -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        // MOVED LOGIN UP: Authentication is now done before building/pulling layers
        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-creds', variable: 'DOCKER_HUB_TOKEN')]) {
                    // Using variables to keep things dynamic
                    bat "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_TOKEN}"
                }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    // This now works because we are logged in
                    bat "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                script {
                    // Timeout increased to 15m to prevent the "deadline exceeded" error
                    bat "docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --timeout 15m --scanners vuln --format table -o trivy-image-report.html ${IMAGE_NAME}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push ${IMAGE_NAME}"
            }
        }
        
        stage('Updating EKS Kubeconfig') {
            steps {
                bat "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
                // Ensure credentialsId 'kube' exists in Jenkins Credentials
                withKubeConfig(caCertificate: '', clusterName: "${CLUSTER_NAME}", contextName: '', credentialsId: 'kube', namespace: "${NAMESPACE}", restrictKubeConfigAccess: false, serverUrl: 'https://AB2AD8E7E396070F02E8CEC4D6A0D7E9.gr7.us-east-1.eks.amazonaws.com') {
                    // Windows-friendly image name replacement in the manifest
                    powershell "(Get-Content deployment.yml).replace('replace', '${IMAGE_NAME}') | Set-Content deployment.yml"
                    bat "kubectl apply -f deployment.yml -n ${NAMESPACE}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                bat "kubectl get pods -n ${NAMESPACE}"
                bat "kubectl get svc -n ${NAMESPACE}"
            }
        }
    }

    post {
        always {
            script {
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = (pipelineStatus == 'SUCCESS') ? 'green' : 'red'

                emailext (
                    subject: "${env.JOB_NAME} - Build ${env.BUILD_NUMBER} - ${pipelineStatus}",
                    body: """<html><body>
                             <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                             <h2>${env.JOB_NAME} - Build ${env.BUILD_NUMBER}</h2>
                             <h3 style="color: ${bannerColor};">Status: ${pipelineStatus}</h3>
                             <p>View details here: <a href="${env.BUILD_URL}">Console Output</a></p>
                             </div></body></html>""",
                    to: 'mohamedsadiq9741@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}
