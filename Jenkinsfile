pipeline {
    agent any

    tools {
        NodeJS "NodeJS_16"   // your configured NodeJS in Jenkins
    }

    parameters {
        choice(name: 'COLLECTION', choices: ['Credit Card Processing - Back Office', 'ACH Collection'], description: 'Select Postman collection to run')
        choice(name: 'ENVIRONMENT', choices: ['SuitePayments - Visa - Release QA'], description: 'Select Postman environment')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo-url.git'
            }
        }

        stage('Install Newman & Reporters') {
            steps {
                sh 'npm install -g newman newman-reporter-allure'
            }
        }

        stage('Run Postman Tests') {
    steps {
        sh """
            # delete old allure-results if any
            rm -rf allure-results

            # run only allure reporter to avoid duplicates
            npx newman run "collections/${params.COLLECTION}.postman_collection.json" \
            -e "environments/${params.ENVIRONMENT}.postman_environment.json" \
            --reporters allure \
            --reporter-allure-export "allure-results"
        """
    }
}

        stage('Allure Report') {
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
            archiveArtifacts artifacts: '**/allure-results/**', allowEmptyArchive: true
        }
    }
}
