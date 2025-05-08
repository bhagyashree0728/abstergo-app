// Magic words to stop grammar warnings
@SuppressWarnings(['FieldDefinition', 'MethodReturnTypeRequired'])

pipeline {
    // Tell Jenkins to run in a Docker container (like a playbox)
    agent {
        docker {
            image 'docker:latest' 
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        // Your secret passwords (like locker combinations)
        IMAGE_NAME = 'your-dockerhub-username/abstergo-app'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                // Get the code from GitHub
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image (like making a toy)
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                // Test the code (like checking if toy works)
                sh 'echo "Running tests..."'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub (with your secret password)
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                            docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes (like sending toy to toy store)
                    withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG')]) {
                        sh """
                            kubectl apply -f k8s-deployment.yaml
                            kubectl set image deployment/abstergo-deployment \
                              abstergo-container=${IMAGE_NAME}:${BUILD_NUMBER} --record
                            kubectl rollout status deployment/abstergo-deployment
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Always clean up (like putting toys away)
            sh 'docker logout'
            
            // Send email (like telling mom what happened)
            emailext (
                subject: "Pipeline '${env.JOB_NAME}' (${env.BUILD_NUMBER}) completed",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'dev-team@abstergo.com'
            )
        }
        
        success {
            // If happy, do a dance! (optional)
            echo 'Yay! Pipeline worked!'
        }
        
        failure {
            // If sad, cry a little (optional)
            echo 'Boo! Pipeline failed!'
        }
    }
}
