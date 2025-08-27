pipeline {
    agent any
    parameters {
        string(
            name: 'COLLECTION',
            defaultValue: '',
            description: 'Enter the collection JSON file name from /collections directory'
        )
        string(
            name: 'ENVIRONMENT',
            defaultValue: '',
            description: 'Enter the environment JSON file name from /environments directory'
        )
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('List Available Files') {
            steps {
                script {
                    echo "Available Collections:"
                    bat "dir collections\\*.json"
                    
                    echo "Available Environments:"
                    bat "dir environments\\*.json"
                }
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                    npx newman run "collections/${COLLECTION}" \
                    -e "environments/${ENVIRONMENT}" \
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
}