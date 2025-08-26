pipeline {
    agent any

    tools {
        nodejs "NodeJS"   // 👈 must match the name you gave in Jenkins
    }

    parameters {
        choice(
            name: 'COLLECTION',
            choices: [
                'ACH Processing - Back Office.postman_collection.json',
                'Credit Card Processing - Back Office.postman_collection.json'
            ],
            description: 'Select which Postman Collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: [
                'SuitePayments - Visa - Release QA.postman_environment.json',
                'SuitePayments - Visa - UAT - Vishali.postman_environment.json'
            ],
            description: 'Select which Environment to use'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Install Newman') {
            steps {
                bat "npm install -g newman newman-reporter-htmlextra"
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                newman run collections\\"${params.COLLECTION}" ^
                    -e environments\\"${params.ENVIRONMENT}" ^
                    -r cli,htmlextra --reporter-htmlextra-export newman-report.html
                """
            }
        }

        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'newman-report.html',
                    reportName: "Newman Report - ${params.COLLECTION} (${params.ENVIRONMENT})"
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'newman-report.html', fingerprint: true
        }
    }
}
