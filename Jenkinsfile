pipeline {
    agent { label 'Jenkins-Agent' }
    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_REPO = "wanted14/${APP_NAME}"
        IMAGE_TAG = params.IMAGE_TAG ?: "latest"
        GIT_REPO_URL = "https://github.com/wnteed/gitops-registration-app"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout Repository") {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: '${GIT_REPO_URL}'
            }
        }
        stage("Update Kubernetes Deployment") {
            steps {
                dir('k8s') {
                    sh '''
                        echo "Original deployment.yaml:"
                        cat deployment.yaml
                        sed -i "s|image: .*$|image: ${IMAGE_REPO}:${IMAGE_TAG}|" deployment.yaml
                        echo "Updated deployment.yaml:"
                        cat deployment.yaml
                    '''
                }
            }
        }
        stage("Push Changes to Git") {
            steps {
                script {
                    // Set git configuration
                    sh '''
                        git config --global user.name "Jenkins CI"
                        git config --global user.email "rayane.matloub2@gmail.com"
                    '''
                    
                    // Check if there are changes to commit
                    def hasChanges = sh(script: 'git status --porcelain | wc -l', returnStdout: true).trim().toInteger() > 0
                    
                    if (hasChanges) {
                        // Stage and commit changes
                        sh 'git add k8s/deployment.yaml'
                        sh "git commit -m 'Updated deployment to ${IMAGE_REPO}:${IMAGE_TAG}'"
                        
                        // Push using credentials
                        withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            // Use HTTP basic auth with credentials in the URL
                            sh '''
                                git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/wnteed/gitops-registration-app.git
                                git push origin main
                            '''
                        }
                        echo "Successfully pushed changes to GitOps repository"
                    } else {
                        echo "No changes detected in deployment.yaml"
                    }
                }
            }
        }
        stage("Apply Deployment to Kubernetes") {
            steps {
                withKubeConfig(credentialsId: 'kubeconfig') {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
    post {
        success {
            echo "CD Pipeline completed successfully"
        }
        failure {
            echo "CD Pipeline failed"
        }
    }
}
