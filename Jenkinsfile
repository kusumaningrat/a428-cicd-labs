pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true'
        DEPLOY_APPROVED = 'false' // Shared environment variable
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(
                        message: 'Lanjutkan ke tahap deploy? (Click "Proceed" to continue)',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition', 
                             choices: 'Proceed\nAbort', 
                             description: 'Select an option',
                             name: 'ACTION']
                        ]
                    )
                    
                    if (userInput == 'Proceed') {
                        echo 'Continuing to Deploy stage'
                        DEPLOY_APPROVED = 'true' // Set the environment variable
                    } else {
                        echo 'Aborting the pipeline'
                        currentBuild.result = 'ABORTED'
                        error('Pipeline aborted by user')
                    }
                    echo "DEPLOY_APPROVED: ${DEPLOY_APPROVED}"
                }
            }
        }
        stage('Deploy') {
            when {
                expression { return env.DEPLOY_APPROVED }
            }
            steps {
                script {
                    def remoteCommand = "ls -l /home/"
                    def sshKey = readFile('/home/devops/privkey.pem') 

                    def sshCommand = "ssh -i ${sshKey} ubuntu@13.212.247.57 '${remoteCommand}'"
                    
                    def sshResult = sh(script: sshCommand, returnStatus: true)
                    
                    if (sshResult == 0) {
                        echo "SSH command executed successfully"
                    } else {
                        error "SSH command failed with exit code ${sshResult}"
                    }
                }
                // sh './jenkins/scripts/deliver.sh'
                // sleep(time: 60, unit: 'SECONDS')
                // sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
s