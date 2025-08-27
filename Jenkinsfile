pipeline {
    agent any

    environment {
        COLLECTION_DIR = "collections"
        ENVIRONMENT_DIR = "environments"
    }

    parameters {
        string(name: 'COLLECTION', defaultValue: '', description: 'Enter collection JSON file name (auto-discovered)')
        string(name: 'ENVIRONMENT', defaultValue: '', description: 'Enter environment JSON file name (auto-discovered)')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Discover Files') {
            steps {
                script {
                    def collections = sh(
                        script: "ls ${COLLECTION_DIR}/*.json",
                        returnStdout: true
                    ).trim().split("\n")

                    def environments = sh(
                        script: "ls ${ENVIRONMENT_DIR}/*.json",
                        returnStdout: true
                    ).trim().split("\n")

                    echo "Available collections: ${collections}"
                    echo "Available environments: ${environments}"

                    // If user didn't pass manually, pick the first available
                    if (!params.COLLECTION?.trim()) {
                        env.COLLECTION = collections[0]
                    } else {
                        env.COLLECTION = params.COLLECTION
                    }

                    if (!params.ENVIRONMENT?.trim()) {
                        env.ENVIRONMENT = environments[0]
                    } else {
                        env.ENVIRONMENT = params.ENVIRONMENT
                    }
                }
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                    npx newman run "${env.COLLECTION}" \
                        -e "${env.ENVIRONMENT}" \
                        -r cli,allure --reporter-allure-export "allure-results"
                """
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/allure-results/*', allowEmptyArchive: true
        }
        failure {
            script {
                currentBuild.result = 'FAILURE'
            }
        }
    }
}