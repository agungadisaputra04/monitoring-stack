pipeline {
    agent any

    environment {
        VM101 = 'agung@192.168.50.10'
        CADVISOR_DIR = '/opt/monitoring/agents/cadvisor'
    }

    stages {

        stage('Validate') {
            steps {
                sh '''
                    docker compose \
                      -f agents/cadvisor/docker-compose.yml \
                      config
                '''
            }
        }

        stage('Deploy cAdvisor') {
            steps {
                sshagent(credentials: ['vm101-deploy-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes ${VM101} \
                          "mkdir -p ${CADVISOR_DIR}"

                        scp -o StrictHostKeyChecking=yes \
                          agents/cadvisor/docker-compose.yml \
                          ${VM101}:${CADVISOR_DIR}/docker-compose.yml

                        ssh -o StrictHostKeyChecking=yes ${VM101} \
                          "cd ${CADVISOR_DIR} && docker compose pull && docker compose up -d"
                    '''
                }
            }
        }

        stage('Validate cAdvisor') {
            steps {
                sshagent(credentials: ['vm101-deploy-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes ${VM101} \
                          "docker ps --filter name=monitoring-cadvisor"

                        ssh -o StrictHostKeyChecking=yes ${VM101} \
                          "curl -fsS http://localhost:8080/metrics > /dev/null"
                    '''
                }
            }
        }
    }
}