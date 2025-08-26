pipeline {
    agent any

    tools {
        NodeJS "NodeJS"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:your-org/your-repo.git',
                    credentialsId: 'your-ssh-key'
            }
        }

        stage('Run Postman Tests') {
            steps {
                sh '''
                    # remove old reports to avoid duplicates
                    rm -rf allure-results allure-report

                    # run Newman ONCE
                    npx newman run "collections/Credit Card Processing - Back Office.postman_collection.json" \
                        -e "environments/SuitePayments - Visa - Release QA.postman_environment.json" \
                        -r allure --reporter-allure-export "allure-results"
                '''
            }
        }

        stage('Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'allure-results']]
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true
        }
    }
}