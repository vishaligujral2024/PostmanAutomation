pipeline {
    agent any
    tools {
        NodeJS "NodeJS"  
    }

    stages {
        stage('Clean old Allure folders') {
            steps {
                script {
                    // Clean allure-results and allure-report folders before every run
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