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

        stage('Build') {
            steps {
                bat "mvn package"
            }
        }

        stage('sonarqube-stage'){
            steps {
                // Fixed the double "sh" issue and used bat
                bat "mvn sonar:sonar -Dsonar.projectKey=devops -Dsonar.host.url=http://localhost:9000 -Dsonar.login=squ_99e2a8a32fb79e14301b4442e0e0db4cda36728b"
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    bat "docker build -t manojkrishnappa/fullstack:${GIT_COMMIT} ."
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                script {
                    // Ensure trivy is in your Windows Environment Path
                    bat "trivy image --format table -o trivy-image-report.html manojkrishnappa/fullstack:${GIT_COMMIT}"
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // On Windows, use double quotes for the echo command
                        bat "echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat "docker push manojkrishnappa/fullstack:${GIT_COMMIT}"
                }
            }
        }
        
        stage('Updating the Cluster') {
            steps {
                script {
                    bat "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"
                }
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-complete-eks', contextName: '', credentialsId: 'kube', namespace: 'sadiq', restrictKubeConfigAccess: false, serverUrl: 'https://AB2AD8E7E396070F02E8CEC4D6A0D7E9.gr7.us-east-1.eks.amazonaws.com') {
                    // Windows does not have 'sed' by default. 
                    // If you have Git Bash installed, 'sh' will work for sed, otherwise use PowerShell:
                    powershell "(Get-Content deployment.yml).replace('replace', '${IMAGE_NAME}') | Set-Content deployment.yml"
                    bat "kubectl apply -f deployment.yml -n %NAMESPACE%"
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-complete-eks', contextName: '', credentialsId: 'kube', namespace: 'sadiq', restrictKubeConfigAccess: false, serverUrl: 'https://AB2AD8E7E396070F02E8CEC4D6A0D7E9.gr7.us-east-1.eks.amazonaws.com') {
                    bat "kubectl get pods -n %NAMESPACE%"
                    bat "kubectl get svc -n %NAMESPACE%"
                }
            }
        }
    }
