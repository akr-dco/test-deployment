pipeline {
    agent any

    environment {
        TARGET_USER = "onprem"
        TARGET_HOST = "192.168.192.66"
    }

    stages {

        stage('Resolve Target Directory') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.TARGET_DIR = "/home/onprem/cicd-prod"
                    } else if (env.BRANCH_NAME == 'staging') {
                        env.TARGET_DIR = "/home/onprem/cicd-testing"
                    } else {
                        error "Branch ${env.BRANCH_NAME} is not allowed to deploy"
                    }
                }

                echo "Branch        : ${env.BRANCH_NAME}"
                echo "Deploy target : ${TARGET_USER}@${TARGET_HOST}:${env.TARGET_DIR}"
            }
        }

        stage('Prepare Target') {
            steps {
                sshagent(['jenkins-onprem']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        mkdir -p ${TARGET_DIR}
                    '
                    """
                }
            }
        }

        stage('Sync Repository') {
            steps {
                sshagent(['jenkins-onprem']) {
                    sh """
                    rsync -avz --delete \
                        --exclude '.git' \
                        --exclude '.jenkins' \
                        ./ \
                        ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}/
                    """
                }
            }
        }

        stage('Deploy Docker Compose') {
            steps {
                sshagent(['jenkins-onprem']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        cd ${TARGET_DIR} &&
                        docker compose pull &&
                        docker compose up -d
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS for ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Deployment FAILED for ${env.BRANCH_NAME}"
        }
    }
}
