// Declarative Pipeline
pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true'
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
                    DEPLOY_APPROVED = 'true'
                    sleep(time: 60, unit: 'SECONDS')
                } else {
                    echo 'Aborting the pipeline'
                    currentBuild.result = 'ABORTED'
                    error('Pipeline aborted by user')
                }
            }
        }
        stage('Deploy') {
            when {
                expression { return env.DEPLOY_APPROVED == 'true' }
            }
            steps {
                echo 'Running Deploy stage'
                sh './jenkins/scripts/deliver.sh'
                sleep(time: 60, unit: 'SECONDS')
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}