pipeline {
    agent any

    environment {
        TARGET_USER = "onprem"
        TARGET_HOST = "192.168.192.66"
        TARGET_DIR  = "/home/onprem/cicd-testing"
    }

    stages {

        stage('Info') {
            steps {
                echo "Branch        : ${env.BRANCH_NAME}"
                echo "Deploy target : ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}"
            }
        }

        stage('Prepare Target') {
            steps {
                sshagent(['privatekey-akr']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        mkdir -p ${TARGET_DIR}
                    '
                    """
                }
            }
        }

        stage('Sync docker-compose') {
            steps {
                sshagent(['privatekey-akr']) {
                    sh """
                    scp -o StrictHostKeyChecking=no docker-compose.yml \
                        ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}/docker-compose.yml
                    """
                }
            }
        }

        stage('Deploy Docker Compose') {
            steps {
                sshagent(['privatekey-akr']) {
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
            echo "✅ Deployment SUCCESS"
        }
        failure {
            echo "❌ Deployment FAILED"
        }
    }
}
