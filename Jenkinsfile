pipeline {
    agent any
    
    environment {
        // Reference the credentials we added earlier
        DOCKER_HUB = credentials('docker-hub-credentials')
        IMAGE_NAME = 'your-dockerhub-username/abstergo-app'
        KUBE_CONFIG = credentials('kube-config')
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                sh 'git branch'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build with build number as tag
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                // Add your test commands here
                sh 'echo "Running tests..."'
                // Example: sh 'npm test' or sh './gradlew test'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    
                    // Push the image
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    
                    // Also tag as latest and push
                    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Write kubeconfig to file
                    writeFile file: "${env.HOME}/.kube/config", text: "${KUBE_CONFIG}"
                    sh "chmod 600 ${env.HOME}/.kube/config"
                    
                    // Apply Kubernetes manifests
                    sh "kubectl apply -f k8s-deployment.yaml"
                    
                    // Update image in deployment
                    sh """
                        kubectl set image deployment/abstergo-deployment \
                        abstergo-container=${IMAGE_NAME}:${BUILD_NUMBER} --record
                    """
                    
                    // Verify deployment
                    sh "kubectl rollout status deployment/abstergo-deployment"
                }
            }
        }
    }
    
    post {
        always {
            // Clean up Docker credentials
            sh 'docker logout'
            
            // Send notification (optional)
            emailext (
                subject: "Pipeline '${env.JOB_NAME}' (${env.BUILD_NUMBER}) completed",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'dev-team@abstergo.com'
            )
        }
        
        success {
            // Additional success actions
            slackSend(color: 'good', message: "Pipeline SUCCESS: ${env.JOB_NAME} - ${env.BUILD_NUMBER}")
        }
        
        failure {
            // Additional failure actions
            slackSend(color: 'danger', message: "Pipeline FAILED: ${env.JOB_NAME} - ${env.BUILD_NUMBER}")
        }
    }
}
