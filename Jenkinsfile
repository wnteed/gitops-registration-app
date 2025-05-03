pipeline {
    agent { label 'Jenkins-Agent' }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
    }

    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_REPO = "wanted14/${APP_NAME}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout Repository") {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/wnteed/gitops-registration-app'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat k8s/deployment.yaml
                    sed -i 's|${APP_NAME}:.*|${APP_NAME}:${params.IMAGE_TAG}|g' k8s/deployment.yaml
                    cat k8s/deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "Jenkins CI"
                    git config --global user.email "rayane.matloub2@gmail.com"
                    git add k8s/deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                """

                withCredentials([gitUsernamePassword(credentialsId: 'github-token', gitToolName: 'Default')]) {
                    sh "git push https://github.com/wnteed/gitops-registration-app main"
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
