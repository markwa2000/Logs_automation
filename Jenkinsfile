pipeline {
    agent any
    
     parameters {
        choice(name: 'MICROSERVICE', choices: ['LocationService', 'ProductService', 'OrderService', 'CartServices', 'CustomerService', 'SettingService'], description: 'Select Microservice')
    }

    environment {
        SERVER_HOST = "{SERVERIP}"
        SERVER_USER = "{SERVERUSERNAME}"
        LOGS_BASE_PATH = "{LOGS_PATH}/${params.MICROSERVICE}"
    }

    stages {
        stage('Fetch Logs') {
            steps {
                script {
                    // Fetch logs from each microservice directory
                    sh "ssh -o StrictHostKeyChecking=yes -o UserKnownHostsFile=~/.ssh/known_hosts ${SERVER_USER}@${SERVER_HOST} 'find ${LOGS_BASE_PATH} -type f -name \"*.log\" -exec dirname {} \\;' | sort -u > microservice_directories.txt"

                    // Read each microservice directory and copy logs
                    sh 'while IFS= read -r dir; do scp -o StrictHostKeyChecking=yes -o UserKnownHostsFile=~/.ssh/known_hosts -r ${SERVER_USER}@${SERVER_HOST}:${dir} .; done < microservice_directories.txt'
              
                    // Create the 'all_logs' directory
                    // sh 'mkdir all_logs'

                    // Find all log files and move them to the 'all_logs' directory
                    // sh 'find -type f -name "*.log" -exec cp {} all_logs/ \\;'
                   
                    // Fetch logs for the selected microservice
                    // sh "scp -o StrictHostKeyChecking=yes -o UserKnownHostsFile=~/.ssh/known_hosts -r ${SERVER_USER}@${SERVER_HOST}:\$(find /datadisk/ordrio/node-backend/Logs/${params.MICROSERVICE}/ -type f -name '*.log' -exec echo {} +) ."
                }
            }
        }

        stage('Publish HTML Reports') {
            steps {
                script {
                    def logFilesExist = sh(script: 'find -type f -name "*.log"', returnStatus: true) == 0
                    if (logFilesExist) {
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: '.',
                            reportFiles: '*.log',
                            reportName: 'Microservice Logs'
                        ])
                    } else {
                        echo 'No log files found to display.'
                    }
                }
            }
        }
          stage('Archive Logs') {
             steps {
                 archiveArtifacts artifacts: '*/**/*.log', allowEmptyArchive: true
             }
         }
    }
    post {
      always {
        // Clean the workspace after the build
        cleanWs()
        }
    }
}