pipeline {
    agent any

    parameters {
        choice(name: 'COLLECTION', choices: sh(script: 'ls collections/*.json', returnStdout: true).trim().split('\n'), description: 'Select Postman Collection')
        choice(name: 'ENVIRONMENT', choices: sh(script: 'ls environments/*.json', returnStdout: true).trim().split('\n'), description: 'Select Postman Environment')
    }

    tools {
        NodeJS "NodeJS"   // your configured NodeJS installation in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                script {
                    // Cleanup allure-results if any left over from older runs
                    bat '''
                        rmdir /s /q allure-results || echo no allure-results
                        mkdir allure-results
                        npx newman run "%WORKSPACE%/collections/${params.COLLECTION}" ^
                            -e "%WORKSPACE%/environments/${params.ENVIRONMENT}" ^
                            -r cli,allure --reporter-allure-export "allure-results/build-%BUILD_NUMBER%"
                    '''
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: true, jdk: '', results: [[path: "allure-results/build-${env.BUILD_NUMBER}"]]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-report/**', fingerprint: true
        }
    }
}