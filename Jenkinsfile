pipeline {
    agent { label "Jenkins-Agent" }
    environment {
        APP_NAME = "register-app-pipeline"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCN") {
            steps {
                git branch: 'main', credentialId: 'github', url: 'https://github.com/wnteed/gltops-registration-app'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh '''
                    cat deployment.yaml
                    sed -i 's/$(APP_NAME).*/$(APP_NAME):$(IMAGE_TAG)/g' deployment.yaml
                    cat deployment.yaml
                '''
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh '''
                    git config --global user.name "wnteed"
                    git config --global user.email "rayane.matloub2@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                '''
                withCredentials([gitUsernamePassword(credentialsId: 'github-token', gitToolName: 'Default')]) {
                    sh "git push https://github.com/wnteed/gitops-registration-app main"
                }
            }
        }
    }
}