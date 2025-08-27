pipeline {
    agent any

    tools {
        nodejs "NodeJS"
    }

    parameters {
        choice(name: 'COLLECTION', choices: getFileList('collections'), description: 'Select Postman Collection')
        choice(name: 'ENVIRONMENT', choices: getFileList('environments'), description: 'Select Postman Environment')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                    set RESULTS_DIR=allure-results-%BUILD_NUMBER%
                    if exist %RESULTS_DIR% rmdir /s /q %RESULTS_DIR%
                    npx newman run "%WORKSPACE%/${params.COLLECTION}" ^
                        -e "%WORKSPACE%/${params.ENVIRONMENT}" ^
                        -r cli,allure --reporter-allure-export "%RESULTS_DIR%" ^
                        --suppress-exit-code 1
                """
            }
        }

        stage('Generate Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: "allure-results-%BUILD_NUMBER%"]]
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results-%BUILD_NUMBER%/**', fingerprint: true
        }
    }
}

@NonCPS
def getFileList(folder) {
    def files = []
    new File(folder).eachFile { file ->
        if (file.name.endsWith('.json')) {
            files << "${folder}/${file.name}"
        }
    }
    return files.join('\n')
}