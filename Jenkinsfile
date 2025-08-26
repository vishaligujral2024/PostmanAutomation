pipeline {
    agent any
    tools {
        NodeJS "NodeJS"      }

    parameters {
        choice(name: 'COLLECTION', choices: ['CreditCard.postman_collection.json', 'ACH.postman_collection.json'], description: 'Select Postman collection to run')
        choice(name: 'ENVIRONMENT', choices: ['Dev.postman_environment.json', 'QA.postman_environment.json'], description: 'Select Postman environment to run')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Run Postman Collection') {
            steps {
                script {
                    // Run Newman once with allure reporter
                    sh """
                        newman run "${params.COLLECTION}" \
                        -e "${params.ENVIRONMENT}" \
                        -r allure --reporter-allure-export "allure-results"
                    """
                }
            }
        }

        stage('Generate Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: "allure-results"]]
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/allure-report/**', allowEmptyArchive: true
        }
    }
}