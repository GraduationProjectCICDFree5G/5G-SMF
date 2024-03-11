pipeline {
    agent any
    triggers {
    githubPush()
  }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub')
        }

    stages {
        stage('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }

        stage('Login to Dockerhub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

   
        stage('Clone GitHub repository') {
            steps {
                script {
                    // Clone your GitHub repository
                    git 'https://github.com/GraduationProjectCICDFree5G/5G-SMF.git'
                }
            }
        }

        stage('docker build') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the cloned repository
                    sh """
                        cd 5G-SMF  # Change to your repository directory
                        docker images -a
                        docker build -t 5ggraduationproject/5g-nrf:latest .
                        docker images -a
                    """
                }
            }
        }
    

        stage('Scan Image for Common Vulnerabilities and Exposures') {
            steps {
                sh 'trivy image 5ggraduationproject/5g-smf --output trivy-report.json'
            }
        }
        stage('Pushing to Dockerhub') {
            steps {
                sh 'docker push 5ggraduationproject/5g-smf:latest'
            }
        }

        stage('Build and Package Helm Chart') {
            steps {
                sh 'helm package ./helm/'
            }
        }

        stage('Configure Kubernetes Context') {
            steps {
                sh 'aws eks --region us-east-1 update-kubeconfig --name 9G-Core-Net'
            }
        }

        stage('Deploy Helm Chart on EKS') {
            steps {
                sh 'helm upgrade --install smf ./helm/'
            }
        }
    }

    post {
        always {
            // Archiving Test Result
            archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            sh 'docker rmi 5ggraduationproject/5g-smf'
            sh 'docker rmi 5ggraduationproject/smf-base'
        }
    }
}