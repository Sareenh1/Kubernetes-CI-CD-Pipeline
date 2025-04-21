pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sareenh1/Kubernetes-CI-CD-Pipeline.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    docker.build("your-dockerhub-username/sample-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        docker.image("your-dockerhub-username/sample-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update the deployment with the new image
                    sh "sed -i 's|<your-dockerhub-username>/sample-app:latest|your-dockerhub-username/sample-app:${env.BUILD_ID}|g' k8s-deployment.yaml"
                    
                    // Apply the deployment
                    sh "kubectl apply -f k8s-deployment.yaml"
                    
                    // Verify deployment
                    sh "kubectl rollout status deployment/sample-app"
                }
            }
        }
    }
    
    post {
        success {
            slackSend(color: "good", message: "Pipeline SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
        failure {
            slackSend(color: "danger", message: "Pipeline FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }
}
