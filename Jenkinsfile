pipeline {
    agent { label "Jenkins-Agent" }
    environment {
        APP_NAME = "register-app-pipeline"
    }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Image tag to deploy')
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/wnteed/gitops-registration-app'
            }
        }
        stage("Update the Deployment Tags") {
            steps {
                script {
                    // Let's first check what's in the directory structure
                    sh "ls -la"
                    
                    // Look specifically in the k8s directory
                    sh "ls -la k8s/"
                    
                    // Check any YAML files in the k8s directory
                    sh "find k8s -type f -name '*.yaml' -exec cat {} \\;"
                    
                    // Modify deployment.yaml to use the correct image tag
                    sh """
                        find k8s -type f -name 'deployment.yaml' -exec sed -i 's|${APP_NAME}:.*|${APP_NAME}:${params.IMAGE_TAG}|g' {} \\;
                        
                        # Verify the changes
                        find k8s -type f -name 'deployment.yaml' -exec cat {} \\;
                    """
                    
                    // Fix the app path issue if it exists in any YAML file
                    sh """
                        # Fix any absolute paths in YAML files
                        find k8s -type f -name '*.yaml' -exec sed -i 's|appPath: /k8s|appPath: k8s|g' {} \\;
                    """
                }
            }
        }
        stage("Push the changed deployment files to Git") {
            steps {
                sh '''
                    git config --global user.name "wnteed"
                    git config --global user.email "rayane.matloub2@gmail.com"
                    git add k8s
                    git commit -m "Updated Deployment Manifests" || echo "No changes to commit"
                '''
                withCredentials([gitUsernamePassword(credentialsId: 'github-token', gitToolName: 'Default')]) {
                    sh "git push https://github.com/wnteed/gitops-registration-app main || echo 'Nothing to push'"
                }
            }
        }
    }
    post {
        success {
            echo "Deployment configuration updated successfully with image tag: ${params.IMAGE_TAG}"
        }
        failure {
            echo "Failed to update deployment configuration"
        }
    }
}
