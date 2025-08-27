pipeline {
    agent any

    parameters {
        choice(
            name: 'COLLECTION',
            choices: [
                'ACH Processing - Back Office.postman_collection.json',
                'Credit Card Processing - Back Office.postman_collection.json'
            ],
            description: 'Select Postman Collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: [
                'SuitePayments - Visa - Release QA.postman_environment.json',
                'SuitePayments - Visa - UAT - Vishali.postman_environment.json'
            ],
            description: 'Select Postman Environment to run'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                npx newman run "collections/${COLLECTION}" ^
                  -e "environments/${ENVIRONMENT}" ^
                  -r cli,allure --reporter-allure-export "%WORKSPACE%\\allure-results"
                """
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }
        unsuccessful {
            script {
                currentBuild.result = 'FAILURE'
            }
        }
    }
}