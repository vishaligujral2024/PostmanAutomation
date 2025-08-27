pipeline {
    agent any

    parameters {
        choice(name: 'COLLECTION', choices: [
            'Credit Card Processing - Back Office.postman_collection.json',
            'ACH Processing - Back Office.postman_collection.json'
        ], description: 'Select Postman collection to run')

        choice(name: 'ENVIRONMENT', choices: [
            'SuitePayments - Visa - Release QA.postman_environment.json',
            'SuitePayments - MasterCard - Release QA.postman_environment.json'
        ], description: 'Select Postman environment to use')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                script {
                    try {
                        bat """
                            npx newman run "collections/${params.COLLECTION}" ^
                            -e "environments/${params.ENVIRONMENT}" ^
                            -r cli,allure --reporter-allure-export "allure-results"
                        """
                    } catch (Exception err) {
                        // Capture Newman failure, don’t stop pipeline yet
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Add Environment Info') {
            steps {
                script {
                    writeFile file: 'allure-results/environment.properties', text: """
Build_URL=${env.BUILD_URL}
Collection=${params.COLLECTION}
Environment=${params.ENVIRONMENT}
"""
                }
            }
        }

        stage('Allure Report') {
            steps {
                script {
                    // Preserve test history for trend
                    if (fileExists("allure-report/history")) {
                        dir("allure-results") {
                            bat "xcopy ..\\allure-report\\history history /E /I /Y"
                        }
                    }
                }
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }
        failure {
            echo "Some Newman tests failed. Check Allure report for details."
        }
        unstable {
            echo "Some Newman tests failed. Build marked as UNSTABLE but report generated."
            // Force build to red if you want failure instead of unstable
            currentBuild.result = 'FAILURE'
        }
    }
}