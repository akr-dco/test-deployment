pipeline {
    agent any

    environment {
        TARGET_USER = "akr"
        TARGET_HOST = "192.168.192.66"
        TARGET_DIR  = "/home/akr/docker-cicd/${env.BRANCH_NAME}"
    }

    stages {

        stage('Info') {
            steps {
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Deploy target: ${TARGET_HOST}:${TARGET_DIR}"
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['akr']) {
                    sh """
                    echo ">>> Create folder if not exists"
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        mkdir -p ${TARGET_DIR}
                    '

                    echo ">>> Copy docker-compose"
                    scp -o StrictHostKeyChecking=no docker-compose.yml ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}/docker-compose.yml

                    echo ">>> Deploy compose"
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} "
                        cd ${TARGET_DIR} &&
                        docker compose pull &&
                        docker compose up -d
                    "
                    """
                }
            }
        }
    }
}
