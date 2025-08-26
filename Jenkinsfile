pipeline {
    agent any
    tools {
        nodejs "nodejs" 
    }

    stages {
        stage('Clean old Allure folders') {
            steps {
                script {
                    // Clean allure-results and allure-report folders
                    dir('allure-results') {
                        deleteDir()
                    }
                    dir('allure-report') {
                        deleteDir()
                    }
                }
            }
        }

        stage('Run Postman Collection') {
            steps {
                script {
                    // Run Newman and export allure results fresh
                    sh """
                        newman run "${params.COLLECTION}" \
                        -e "${params.ENVIRONMENT}" \
                        -r allure --reporter-allure-export "allure-results"
                    """
                }
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
            archiveArtifacts artifacts: '**/allure-report/**', allowEmptyArchive: true
        }
    }
}