pipeline {
    agent any
    
    environment {
        // Docker configuration
        DOCKER_IMAGE = 'sraavi1/jenkins-image'
        DOCKER_TAG = "jenkins-${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'e3b0deee-5049-4c59-8a5f-3faea31cddd8'
        
        // Build context
        CONTEXT_DIR = 'jenkins'
        
        // Git configuration for manifest update
        GIT_REPO = 'https://github.com/subhash-06/DevOps-Arsenal.git'
        GIT_CREDENTIALS_ID = 'git'
        MANIFEST_PATH = 'k8s/jenkins/deployment.yaml'
        
        
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo ' Checking out code from Git...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo " Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    dir(CONTEXT_DIR) {
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:jenkins-latest
                        """
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo ' Running tests...'
                    sh """
                        # Test if image was built
                        docker images | grep ${DOCKER_IMAGE}
                        
                        # Run container for testing
                        docker run -d --name test-container -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 5
                        
                        # Test health endpoint
                        curl -f http://localhost:8081/health || exit 1
                        
                        # Test main page
                        curl -f http://localhost:8081 || exit 1
                        
                        # Cleanup test container
                        docker stop test-container
                        docker rm test-container
                    """
                    echo ' Tests passed!'
                }
            }
        }
        
        
        stage('Security Scan') {
    steps {
        script {
            echo 'Running Trivy security scan...'
            sh """
                trivy image --severity HIGH,CRITICAL --exit-code 0 ${DOCKER_IMAGE}:${DOCKER_TAG}
                trivy image --severity HIGH,CRITICAL --format json -o trivy-report.json ${DOCKER_IMAGE}:${DOCKER_TAG}
            """
        }
    }
}
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo " Pushing image to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: DOCKER_CREDENTIALS_ID,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_IMAGE}:jenkins-latest
                            docker logout
                        """
                    }
                    echo ' Image pushed successfully!'
                }
            }
        }
        
        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    echo " Updating ${MANIFEST_PATH} with new image tag..."
                    withCredentials([usernamePassword(
                        credentialsId: GIT_CREDENTIALS_ID,
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                            # Configure git
                            git config user.email "jenkins@cicd.com"
                            git config user.name "Jenkins CI"
                            
                            # Update deployment.yaml with new image tag
                            sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' ${MANIFEST_PATH}
                            
                            # Commit and push changes
                            git add ${MANIFEST_PATH}
                            git commit -m "Jenkins: Update image to ${DOCKER_TAG}" || echo "No changes to commit"
                            git push https://${GIT_USER}:${GIT_TOKEN}@github.com/subhash-06/DevOps-Arsenal.git HEAD:main
                        """
                    }
                    echo ' Manifest updated and pushed to Git!'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo ' Waiting for ArgoCD to sync...'
                    sh """
                        echo "ArgoCD will automatically detect the manifest change and deploy"
                        echo "New image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        echo "Check ArgoCD UI at: http://${ARGOCD_SERVER}"
                    """
                    sleep 10
                }
            }
        }
    }
}   