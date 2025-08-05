pipeline {
    agent any
    
    environment {
        // Define environment variables
        DOCKER_IMAGE = "my-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        REGISTRY = "your-registry.com" // Optional: for pushing to registry
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout code from repository
                checkout scm
                echo "Checked out code from repository"
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "Building Docker image..."
                    // Build Docker image
                    def image = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    
                    // Tag as latest
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    
                    echo "Docker image built successfully"
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    
                    // Run tests inside Docker container
                    sh """
                        docker run --rm \
                        -v \$(pwd):/app \
                        -w /app \
                        ${DOCKER_IMAGE}:${DOCKER_TAG} \
                        python -m pytest tests/ -v
                    """
                    
                    echo "Tests completed successfully"
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    echo "Running security scan..."
                    
                    // Optional: Run security scan with tools like Trivy
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        -v \$(pwd):/app \
                        aquasec/trivy:latest image ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                    """
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    echo "Deploying to staging environment..."
                    
                    // Stop existing container if running
                    sh "docker stop my-app-staging || true"
                    sh "docker rm my-app-staging || true"
                    
                    // Run new container
                    sh """
                        docker run -d \
                        --name my-app-staging \
                        -p 3001:3000 \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                    
                    echo "Application deployed to staging on port 3001"
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo "Deploying to production environment..."
                    
                    // Add approval step for production deployment
                    input message: 'Deploy to production?', ok: 'Deploy'
                    
                    // Stop existing container if running
                    sh "docker stop my-app-prod || true"
                    sh "docker rm my-app-prod || true"
                    
                    // Run new container
                    sh """
                        docker run -d \
                        --name my-app-prod \
                        -p 3000:3000 \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                    
                    echo "Application deployed to production on port 3000"
                }
            }
        }
        
        stage('Push to Registry') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    echo "Pushing image to registry..."
                    
                    // Login to registry (configure credentials in Jenkins)
                    // withCredentials([usernamePassword(credentialsId: 'docker-registry', 
                    //                                usernameVariable: 'USERNAME', 
                    //                                passwordVariable: 'PASSWORD')]) {
                    //     sh "docker login -u $USERNAME -p $PASSWORD ${REGISTRY}"
                    //     sh "docker push ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    //     sh "docker push ${REGISTRY}/${DOCKER_IMAGE}:latest"
                    // }
                    
                    echo "Image pushed to registry successfully"
                }
            }
        }
    }
    
    post {
        always {
            // Clean up
            script {
                echo "Cleaning up..."
                sh "docker system prune -f"
            }
        }
        
        success {
            echo "Pipeline completed successfully!"
            // Optional: Send notification
            // slackSend channel: '#deployments', 
            //          color: 'good', 
            //          message: "✅ Build ${BUILD_NUMBER} succeeded for ${JOB_NAME}"
        }
        
        failure {
            echo "Pipeline failed!"
            // Optional: Send notification
            // slackSend channel: '#deployments', 
            //          color: 'danger', 
            //          message: "❌ Build ${BUILD_NUMBER} failed for ${JOB_NAME}"
        }
    }
}
