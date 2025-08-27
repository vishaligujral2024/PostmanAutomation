pipeline {
    agent any

    tools {
        NodeJS "NodeJS"   // your configured NodeJS tool in Jenkins
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        // Placeholder – actual values will be set dynamically in the first stage
        choice(name: 'COLLECTION', choices: ['dummy'], description: 'Select which Postman Collection to run')
        choice(name: 'ENVIRONMENT', choices: ['dummy'], description: 'Select which Environment to use')
    }

    stages {
        stage('Prepare Parameters') {
            steps {
                script {
                    // Read collection files dynamically
                    def collections = sh(
                        script: "ls collections/*.json",
                        returnStdout: true
                    ).trim().split("\n")*.replaceAll(/^collections[\\\\/]/, '')

                    // Read environment files dynamically
                    def environments = sh(
                        script: "ls environments/*.json",
                        returnStdout: true
                    ).trim().split("\n")*.replaceAll(/^environments[\\\\/]/, '')

                    // Update parameters
                    properties([
                        parameters([
                            choice(name: 'COLLECTION', choices: collections, description: 'Select which Postman Collection to run'),
                            choice(name: 'ENVIRONMENT', choices: environments, description: 'Select which Environment to use')
                        ])
                    ])
                }
            }
        }

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
                bat '''
                    echo ===============================
                    echo Cleaning old Allure results
                    echo ===============================
                    if exist allure-results (
                        rmdir /s /q allure-results
                    )
                    if exist allure-report (
                        rmdir /s /q allure-report
                    )

                    echo ===============================
                    echo Running ONLY this collection:
                    echo ${params.COLLECTION}
                    echo Using environment:
                    echo ${params.ENVIRONMENT}
                    echo ===============================

                    npx newman run "collections\\${params.COLLECTION}" ^
                        -e "environments\\${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "allure-results"
                '''
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
