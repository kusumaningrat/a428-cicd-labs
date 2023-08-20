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
                    def remoteCommand = "ls -l /home"

                    sshCommand remote: [
                        host: 13.212.247.57,
                        user: ubuntu,
                        port: 22,
                        key: credentials('devauth')
                    ], command: remoteCommand, failOnError: true
                }
                // sh './jenkins/scripts/deliver.sh'
                // sleep(time: 60, unit: 'SECONDS')
                // sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
s