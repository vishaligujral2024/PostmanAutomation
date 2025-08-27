pipeline {
    agent any

    parameters {
        choice(name: 'COLLECTION', choices: sh(script: "ls collections", returnStdout: true).trim().split("\n"), description: 'Select Postman Collection')
        choice(name: 'ENVIRONMENT', choices: sh(script: "ls environments", returnStdout: true).trim().split("\n"), description: 'Select Environment File')
    }

    stages {
        stage('Run Newman Tests') {
            steps {
                bat """
                npx newman run "collections/${params.COLLECTION}" ^
                    -e "environments/${params.ENVIRONMENT}" ^
                    -r cli,allure --reporter-allure-export "allure-results"
                """
            }
        }
    }

    post {
        always {
            script {
                // Always generate and attach allure report, even if tests fail
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'allure-results']]
                ])
            }
        }
        unsuccessful {
            echo "Some Newman tests failed . Check the Allure Report for details."
        }
        success {
            echo "All Newman tests passed ."
        }
    }
}