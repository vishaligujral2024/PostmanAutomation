pipeline {
    agent any

    tools {
        NodeJS "NodeJS"   // configured NodeJS tool name in Jenkins
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(
            name: 'COLLECTION',
            choices: sh(script: 'dir /b collections\\*.json', returnStdout: true).trim().split("\\r?\\n"),
            description: 'Select Postman Collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: sh(script: 'dir /b environments\\*.json', returnStdout: true).trim().split("\\r?\\n"),
            description: 'Select Postman Environment to use'
        )
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
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
                    echo Running collection: ${params.COLLECTION}
                    echo Using environment: ${params.ENVIRONMENT}
                    echo ===============================

                    set RESULTS_DIR=allure-results\\build-${env.BUILD_NUMBER}
                    if exist %RESULTS_DIR% rmdir /s /q %RESULTS_DIR%

                    npx newman run "collections\\${params.COLLECTION}" ^
                        -e "environments\\${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "%RESULTS_DIR%" ^
                        --disable-unicode
                """
            }
        }

        stage('Add Environment Info') {
            steps {
                script {
                    def envFile = "allure-results/build-${env.BUILD_NUMBER}/environment.properties"
                    writeFile file: envFile, text: """
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
                allure includeProperties: true, jdk: '', results: [[path: "allure-results/build-${env.BUILD_NUMBER}"]]
            }
        }

        stage('Cleanup Old Results') {
            steps {
                bat """
                    echo Cleaning up old allure-results except current build...
                    for /d %%D in (allure-results\\build-*) do (
                        if not "%%~nxD"=="build-${env.BUILD_NUMBER}" (
                            echo Deleting %%D
                            rmdir /s /q "%%D"
                        )
                    )
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "allure-results/build-${env.BUILD_NUMBER}/**", fingerprint: true
        }
    }
}
