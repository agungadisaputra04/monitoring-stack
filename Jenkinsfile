pipeline {
    agent any

    environment {
        VM101 = 'agung@192.168.50.10'
        CADVISOR_DIR = '/opt/monitoring/agents/cadvisor'
    }

    stage('Deploy cAdvisor') {
        steps {
            withCredentials([
                sshUserPrivateKey(
                    credentialsId: 'vm101-deploy-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )
            ]) {
                sh '''
                    ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    "${SSH_USER}@192.168.50.10" \
                    "mkdir -p ${CADVISOR_DIR}"

                    scp -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    agents/cadvisor/docker-compose.yml \
                    "${SSH_USER}@192.168.50.10:${CADVISOR_DIR}/docker-compose.yml"

                    ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    "${SSH_USER}@192.168.50.10" \
                    "cd ${CADVISOR_DIR} && docker compose pull && docker compose up -d"
                '''
            }
        }
    }

    stage('Validate cAdvisor') {
        steps {
            withCredentials([
                sshUserPrivateKey(
                    credentialsId: 'vm101-deploy-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )
            ]) {
                sh '''
                    ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    "${SSH_USER}@192.168.50.10" \
                    "docker ps --filter name=monitoring-cadvisor"

                    ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    "${SSH_USER}@192.168.50.10" \
                    "curl -fsS http://localhost:8080/metrics > /dev/null"
                '''
            }
        }
    }
}