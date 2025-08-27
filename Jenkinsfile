pipeline {
    agent any

    parameters {
        choice(name: 'COLLECTION', choices: sh(script: "ls collections/*.json", returnStdout: true).trim().split("\n"), description: 'Select Postman collection')
        choice(name: 'ENVIRONMENT', choices: sh(script: "ls environments/*.json", returnStdout: true).trim().split("\n"), description: 'Select Postman environment')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                script {
                    // Ensure no duplicate reports stick between builds
                    if (isUnix()) {
                        sh 'rm -rf newman-report.html newman newman/*.json'
                    } else {
                        bat 'if exist newman-report.html del /f /q newman-report.html'
                        bat 'if exist newman rmdir /s /q newman'
                    }
                }
            }
        }

        stage('Run Newman Tests') {
            steps {
                script {
                    // Run tests with htmlextra reporter
                    def status = bat(
                        script: """
                            npx newman run "%WORKSPACE%/${params.COLLECTION}" ^
                                -e "%WORKSPACE%/${params.ENVIRONMENT}" ^
                                -r cli,htmlextra --reporter-htmlextra-export newman-report.html
                        """,
                        returnStatus: true
                    )
                    
                    // Fail the build if Newman had failures
                    if (status != 0) {
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'newman-report.html', fingerprint: true
        }
    }
}