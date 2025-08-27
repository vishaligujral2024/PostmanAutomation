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
    npx newman run ^"collections/${params.COLLECTION}^" ^
    -e ^"environments/${params.ENVIRONMENT}^" ^
    -r cli,allure --reporter-allure-export ^"allure-results^"
"""
                    } catch (Exception err) {
                        // Mark build unstable so Allure report still runs
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
                    // Preserve test history
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
            echo " Newman tests failed. See Allure report."
        }
        unstable {
            echo " Some Newman tests failed. Build marked as UNSTABLE but report generated."
            script {
                // Force build to red if you don’t want yellow
                currentBuild.result = 'FAILURE'
            }
        }
    }
}