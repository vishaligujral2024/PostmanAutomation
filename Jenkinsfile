pipeline {
    agent any

    tools {
        NodeJS "NodeJS_16"
    }

    stages {
        stage('Workspace Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                script {
                    // Extra safeguard: wipe old allure-results if it exists
                    bat 'if exist allure-results rmdir /s /q allure-results'

                    bat '''
                        npx newman run "collections/Credit Card Processing - Back Office.postman_collection.json" ^
                        -e "environments/SuitePayments - Visa - Release QA.postman_environment.json" ^
                        -r cli,allure --reporter-allure-export "allure-results"
                    '''
                }
            }
        }

        stage('Generate Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**, allure-report/**', allowEmptyArchive: true
        }
    }
}