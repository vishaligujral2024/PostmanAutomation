properties([
  parameters([
    // Dynamic collection dropdown
    [$class: 'ChoiceParameter',
      name: 'COLLECTION',
      choiceType: 'PT_SINGLE_SELECT',
      description: 'Select Postman Collection to run',
      script: [$class: 'GroovyScript',
        script: [classpath: [], sandbox: true, script: '''
          def dir = new File("${WORKSPACE}/collections")
          if (!dir.exists()) return ["No collections found"]
          return dir.listFiles()
                   .findAll { it.name.endsWith(".json") }
                   .collect { it.name }
                   .sort()
        '''],
        fallbackScript: [classpath: [], sandbox: true, script: 'return ["Error"]']
      ]
    ],

    // Dynamic environment dropdown
    [$class: 'ChoiceParameter',
      name: 'ENVIRONMENT',
      choiceType: 'PT_SINGLE_SELECT',
      description: 'Select Postman Environment to run',
      script: [$class: 'GroovyScript',
        script: [classpath: [], sandbox: true, script: '''
          def dir = new File("${WORKSPACE}/environments")
          if (!dir.exists()) return ["No environments found"]
          return dir.listFiles()
                   .findAll { it.name.endsWith(".json") }
                   .collect { it.name }
                   .sort()
        '''],
        fallbackScript: [classpath: [], sandbox: true, script: 'return ["Error"]']
      ]
    ]
  ])
])

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaligujral2024/PostmanAutomation.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                bat """
                npx newman run "collections/${COLLECTION}" ^
                  -e "environments/${ENVIRONMENT}" ^
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
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }
        unsuccessful {
            script {
                currentBuild.result = 'FAILURE'
            }
        }
    }
}