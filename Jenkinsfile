pipeline {
    agent any

    tools {
        nodejs "NodeJS"
    }

    parameters {
        choice(
            name: 'COLLECTION',
            choices: ['placeholder'],  // dummy choice
            description: 'Select Postman collection to run'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['placeholder'],  // dummy choice
            description: 'Select Postman environment to run'
        )
    }

    stages {
        stage('Setup Parameters') {
            steps {
                script {
                    // Dynamically list all available collections
                    def collections = sh(
                        script: "ls collections/*.json | xargs -n 1 basename",
                        returnStdout: true
                    ).trim().split("\n")

                    // Dynamically list all environments
                    def environments = sh(
                        script: "ls environments/*.json | xargs -n 1 basename",
                        returnStdout: true
                    ).trim().split("\n")

                    // Update build parameters dynamically
                    properties([
                        parameters([
                            choice(name: 'COLLECTION', choices: collections, description: 'Select Postman collection'),
                            choice(name: 'ENVIRONMENT', choices: environments, description: 'Select Postman environment')
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
                    npx newman run "collections/${params.COLLECTION}" ^
                        -e "environments/${params.ENVIRONMENT}" ^
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