pipeline {
    agent any

    tools {
        nodejs "NodeJS"  // Configured in Jenkins Global Tool Configuration
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

        stage('Install Newman + Allure Reporter') {
            steps {
                bat 'npm install -g newman newman-reporter-allure'
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                    newman run "collections/${params.COLLECTION}" ^
                        -e "environments/${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "allure-results"
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
    }
}
