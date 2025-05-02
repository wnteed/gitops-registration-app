pipeline {
    agent { label 'Jenkins-Agent' }

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

        stage("Push Changes to Git (Optional)") {
            steps {
                script {
                    sh '''
                        git config --global user.name "wnteed"
                        git config --global user.email "rayane.matloub2@gmail.com"
                        git add k8s/deployment.yaml
                        git commit -m "Updated deployment to ${IMAGE_REPO}:${IMAGE_TAG}" || echo "No changes"
                    '''
                    withCredentials([gitUsernamePassword(credentialsId: 'github-token', gitToolName: 'Default')]) {
                        sh "git push https://github.com/wnteed/gitops-registration-app main"
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
}
