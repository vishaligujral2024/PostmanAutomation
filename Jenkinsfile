pipeline {
    agent any

    tools {
        NodeJS "NodeJS_16"
    }

    parameters {
        choice(name: 'COLLECTION', choices: sh(
            script: "ls collections/*.json",
            returnStdout: true
        ).trim().split("\\n"),
        description: 'Select Postman Collection')

        choice(name: 'ENVIRONMENT', choices: sh(
            script: "ls environments/*.json",
            returnStdout: true
        ).trim().split("\\n"),
        description: 'Select Postman Environment')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                bat """
                    rmdir /s /q allure-results || echo "no allure-results"
                    rmdir /s /q allure-report || echo "no allure-report"
                """
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
                    def resultsDir = "allure-results"
                    bat """
                        mkdir "${resultsDir}"
                        npx newman run "%WORKSPACE%/${params.COLLECTION}" ^
                            -e "%WORKSPACE%/${params.ENVIRONMENT}" ^
                            -r cli,allure --reporter-allure-export "${resultsDir}"
                        rmdir /s /q "${resultsDir}\\.history" || echo "no history"
                    """
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: "allure-results"]],
                       reusePreviousBuild: false
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**, allure-report/**', allowEmptyArchive: true
        }
    }
}