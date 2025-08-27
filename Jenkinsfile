pipeline {
    agent any

    tools {
        NodeJS "NodeJS"   // your configured NodeJS installation
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(name: 'COLLECTION', choices: ['default_collection.json'], description: 'Select Postman Collection')
        choice(name: 'ENVIRONMENT', choices: ['default_environment.json'], description: 'Select Environment')
        booleanParam(name: 'REFRESH_PARAMS', defaultValue: false, description: 'Check to refresh available parameters')
    }

    stages {
        stage('Checkout & Hard Clean') {
            steps {
                cleanWs()  // Jenkins plugin cleanup
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'

                bat '''
                    echo === HARD CLEAN START ===
                    if exist allure-results rmdir /s /q allure-results
                    if exist allure-report rmdir /s /q allure-report
                    if exist node_modules rmdir /s /q node_modules
                    if exist package-lock.json del /f /q package-lock.json
                    if exist .newman rmdir /s /q .newman
                    echo === HARD CLEAN END ===
                '''
            }
        }

        stage('Prepare Parameters (optional refresh)') {
            when {
                expression { return params.REFRESH_PARAMS?.toBoolean() }
            }
            steps {
                script {
                    def collections = bat(
                        script: 'dir /b collections\\*.json',
                        returnStdout: true
                    ).trim().split("\r?\n")

                    def environments = bat(
                        script: 'dir /b environments\\*.json',
                        returnStdout: true
                    ).trim().split("\r?\n")

                    properties([
                        parameters([
                            choice(name: 'COLLECTION', choices: collections, description: 'Select Postman Collection'),
                            choice(name: 'ENVIRONMENT', choices: environments, description: 'Select Environment'),
                            booleanParam(name: 'REFRESH_PARAMS', defaultValue: false, description: 'Check to refresh available parameters')
                        ])
                    ])
                }
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat '''
                    echo ===============================
                    echo Running collection: ${params.COLLECTION}
                    echo Using environment: ${params.ENVIRONMENT}
                    echo ===============================

                    npx newman run "collections\\${params.COLLECTION}" ^
                        -e "environments\\${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "allure-results" ^
                        --disable-unicode
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
        cleanup {
            bat '''
                echo === FINAL CLEANUP ===
                if exist allure-results rmdir /s /q allure-results
                if exist allure-report rmdir /s /q allure-report
                if exist .newman rmdir /s /q .newman
                echo === CLEANUP COMPLETE ===
            '''
        }
    }
}
