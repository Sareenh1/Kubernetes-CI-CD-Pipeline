pipeline {
    agent none // No default agent; specify per stage
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                git branch: 'main', url: 'https://github.com/Sareenh1/Kubernetes-CI-CD-Pipeline.git'
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'docker:20.10'
                    args '-v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
                }
            }
            steps {
                script {
                    docker.build("sareen/sample-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Test') {
            agent {
                docker { image 'node:14' } // Use Node.js image for testing
            }
            steps {
                sh 'npm install' // Install dependencies before running tests
                sh 'npm test'
            }
        }
        
        stage('Push to Docker Hub') {
            agent {
                docker {
                    image 'docker:20.10'
                    args '-v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                            docker.image("sareen/sample-app:${env.BUILD_ID}").push()
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            agent {
                docker { image 'bitnami/kubectl:latest' }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        // Update the deployment with the new image
                        sh "sed -i 's|sareen/sample-app:latest|sareen/sample-app:${env.BUILD_ID}|g' k8s-deployment.yaml"
                        
                        // Apply the deployment
                        sh "kubectl apply -f k8s-deployment.yaml"
                        
                        // Verify deployment
                        sh "kubectl rollout status deployment/sample-app"
                    }
                }
            }
        }
    }
}
