pipeline {
    agent any

    environment {
        GIT_REPO    = 'https://github.com/nagendar47-hash/testing.git'
        GIT_BRANCH  = 'main'
        IMAGE_NAME  = 'alpine'
        IMAGE_TAG   = '3.19'
        K8S_FILE    = 'k8s/deployment.yaml'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: "${GIT_BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${GIT_REPO}"
            }
        }

        stage('Update Image Tag') {
            steps {
                sh """
                    sed -i 's|image: alpine:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' ${K8S_FILE}
                    cat ${K8S_FILE}
                """
            }
        }

        stage('Push Updated YAML to Git') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins"
                        git add ${K8S_FILE}
                        git commit -m "Updated image to ${IMAGE_NAME}:${IMAGE_TAG}" || echo "No changes"
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/<your-username>/<your-repo>.git ${GIT_BRANCH}
                    """
                }
            }
        }

        stage('ArgoCD Sync') {
            steps {
                sh """
                    argocd login 192.168.64.8:31339 \
                        --username admin \
                        --password NGcmvcJkA1K0TAQW \
                        --insecure
                    argocd app sync alpine-app
                    argocd app wait alpine-app --health
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
