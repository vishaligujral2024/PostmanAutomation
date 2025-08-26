pipeline {
    agent any

    tools {
        nodejs "NodeJS"  // Configured in Jenkins Global Tool Configuration
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(
            name: 'COLLECTION',
            choices: [
                'ACH Processing - Back Office.postman_collection.json',
                'Credit Card Processing - Back Office.postman_collection.json'
            ],
            description: 'Select which Postman Collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: [
                'SuitePayments - Visa - Release QA.postman_environment.json',
                'SuitePayments - Visa - UAT - Vishali.postman_environment.json'
            ],
            description: 'Select which Environment to use'
        )
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()  // ✅ Now plugin handles cleanup
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

       stage('Run Newman Tests') {
    steps {
        bat """
            echo ===============================
            echo Running ONLY this collection:
            echo ${params.COLLECTION}
            echo Using environment:
            echo ${params.ENVIRONMENT}
            echo ===============================

            if exist allure-results rmdir /s /q allure-results
            if exist allure-report rmdir /s /q allure-report

            newman run "collections\\${params.COLLECTION}" ^
                -e "environments\\${params.ENVIRONMENT}" ^
                -r cli,allure --reporter-allure-export "allure-results"
        """
    }
}

        stage('Add Environment Info') {
            steps {
                script {
                    writeFile file: 'allure-results/environment.properties', text: """
                    Collection=${params.COLLECTION}
                    Environment=${params.ENVIRONMENT}
                    Jenkins_Build=${env.BUILD_NUMBER}
                    Jenkins_Job=${env.JOB_NAME}
                    Jenkins_URL=${env.BUILD_URL}
                    """
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: true, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }
    }
}
