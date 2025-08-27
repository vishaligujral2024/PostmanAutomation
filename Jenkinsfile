pipeline {
    agent any

    tools {
        NodeJS "NodeJS"
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
        stage('Deep Clean Workspace') {
            steps {
                cleanWs(deleteDirs: true)

                bat """
                    echo Cleaning leftover dirs...
                    if exist allure-results rmdir /s /q allure-results
                    if exist allure-report rmdir /s /q allure-report
                    if exist target rmdir /s /q target
                    if exist build rmdir /s /q build
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
                    // Unique results folder per build
                    def resultsDir = "allure-results-${env.BUILD_NUMBER}"

                    bat """
                        echo Creating fresh results dir: ${resultsDir}
                        if exist ${resultsDir} rmdir /s /q ${resultsDir}
                        mkdir ${resultsDir}

                        echo Running Newman...
                        npx newman run "%WORKSPACE%/${params.COLLECTION}" ^
                            -e "%WORKSPACE%/${params.ENVIRONMENT}" ^
                            -r cli,allure --reporter-allure-export "${resultsDir}"
                    """

                    // Debug: list generated files
                    bat "dir /b ${resultsDir}"
                }
            }
        }

        stage('Allure Report') {
            steps {
                // Use the unique folder for this build
                allure includeProperties: false, jdk: '', results: [[path: "allure-results-${env.BUILD_NUMBER}"]],
                       reusePreviousBuild: false, keepOnlyLatestBuild: true
            }
        }
    }

    post {
        always {
            echo "Archiving results..."
            archiveArtifacts artifacts: "allure-results-${env.BUILD_NUMBER}/**, allure-report/**", allowEmptyArchive: true

            echo "Cleaning unique results dir..."
            bat "if exist allure-results-${env.BUILD_NUMBER} rmdir /s /q allure-results-${env.BUILD_NUMBER}"
        }
    }
}