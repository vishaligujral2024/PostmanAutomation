pipeline {
    agent any

    tools {
        nodejs "NodeJS"
    }

    parameters {
        choice(
            name: 'COLLECTION',
            choices: ['placeholder'],  // dummy
            description: 'Select Postman collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['placeholder'],  // dummy
            description: 'Select Postman environment to run'
        )
    }

    stages {
        stage('Setup Parameters') {
            steps {
                script {
                    // List collections on Windows
                    def collections = bat(
                        script: 'dir /b collections\\*.json',
                        returnStdout: true
                    ).trim().split("\r?\n")

                    // List environments on Windows
                    def environments = bat(
                        script: 'dir /b environments\\*.json',
                        returnStdout: true
                    ).trim().split("\r?\n")

                    // Update Jenkins build params dynamically
                    properties([
                        parameters([
                            choice(name: 'COLLECTION', choices: collections.join('\n'), description: 'Select Postman collection'),
                            choice(name: 'ENVIRONMENT', choices: environments.join('\n'), description: 'Select Postman environment')
                        ])
                    ])
                }
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
                    npx newman run "collections\\${params.COLLECTION}" ^
                        -e "environments\\${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "allure-results"
                """
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
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }
        unsuccessful {
            echo "Build failed but Allure report still generated!"
        }
    }
}