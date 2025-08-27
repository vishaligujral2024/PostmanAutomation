pipeline {
    agent any

    tools {
        NodeJS "NodeJS_16"   // your configured NodeJS tool in Jenkins
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
                    def resultsDir = "allure-results/build-${env.BUILD_NUMBER}"

                    // Full clean to avoid duplicates
                    bat """
                        rmdir /s /q allure-results || echo "no old allure-results"
                        mkdir "${resultsDir}"
                        npx newman run "%WORKSPACE%/${params.COLLECTION}" ^
                            -e "%WORKSPACE%/${params.ENVIRONMENT}" ^
                            -r cli,allure --reporter-allure-export "${resultsDir}"
                        rmdir /s /q "${resultsDir}\\.history" || echo "no history folder"
                    """
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: "allure-results/build-${env.BUILD_NUMBER}"]]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**, allure-report/**', allowEmptyArchive: true
        }
    }
}