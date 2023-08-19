// Declarative Pipeline
// pipeline {
//     agent {
//         docker {
//             image 'node:lts-buster-slim'
//             args '-p 3000:3000'
//         }
//     }
//     environment {
//         CI = 'true'
//     }
//     stages {
//         stage('Build') {
//             steps {
//                 sh 'npm install'
//             }
//         }
//         stage('Test') {
//             steps {
//                 sh './jenkins/scripts/test.sh'
//             }
//         }
//         stage('Deliver') {
//             steps {
//                 sh './jenkins/scripts/deliver.sh'
//                 input message: 'Finished using the website? (Click "Proceed" to continue)'
//                 sh './jenkins/scripts/kill.sh'
//             }
//         }
//     }
// }

// Scripted Pipeline
node {
    def nodeImage = 'node:14-alpine'
    def npmCache = "${JENKINS_HOME}/.npm-cache"

    stage('Checkout') {
        checkout scm
    }

    stage('Setup') {
        docker.image(nodeImage).inside("-v ${npmCache}:/root/.npm") {
            sh 'npm install'
        }
    }

    stage('Test') {
        docker.image(nodeImage).inside("-v ${npmCache}:/root/.npm") {
            sh './jenkins/scripts/test.sh'
        }
    }

    stage('Deliver') {
        docker.image(nodeImage).inside("-v ${npmCache}:/root/.npm") {
            sh './jenkins/scripts/deliver.sh'
            input message: 'Finished using the website? (Click "Proceed" to continue)'
            sh './jenkins/scripts/kill.sh'
        }
    }