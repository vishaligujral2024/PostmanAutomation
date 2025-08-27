pipeline {
    agent any

    tools {
        nodejs "NodeJS_"     
}

    parameters {
        choice(
            name: 'COLLECTION',
            choices: [
                'Credit Card Processing - Back Office.postman_collection.json',
                'ACH Processing - Back Office.postman_collection.json'
            ],
            description: 'Select Postman collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: [
                'SuitePayments - Visa - Release QA.postman_environment.json',
                'SuitePayments - MasterCard - Release QA.postman_environment.json'
            ],
            description: 'Select Postman environment to run'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                script {
                    bat """
                        npx newman run "collections/${params.COLLECTION}" \
                        -e "environments/${params.ENVIRONMENT}" \
                        -r cli,allure --reporter-allure-export "allure-results"
                    """
                }
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
            archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true
        }
        unsuccessful {
            error("Some Newman tests failed. Check Allure report for details.")
        }
    }
}
