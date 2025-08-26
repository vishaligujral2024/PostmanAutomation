pipeline {
    agent any

    tools {
        NodeJS "NodeJS_16"
    }

    stages {
        stage('Workspace Cleanup') {
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
        bat """
            echo Cleaning old results...
            if exist allure-results rmdir /s /q allure-results
            if exist allure-report rmdir /s /q allure-report

            echo ===============================
            echo Running ONLY this collection:
            echo ${params.COLLECTION}
            echo Using environment:
            echo ${params.ENVIRONMENT}
            echo ===============================

            npx newman run "collections\\${params.COLLECTION}" ^
                -e "environments\\${params.ENVIRONMENT}" ^
                -r cli,allure --reporter-allure-export "allure-results"
        """
    }
}
        stage('Generate Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**, allure-report/**', allowEmptyArchive: true
        }
    }
}