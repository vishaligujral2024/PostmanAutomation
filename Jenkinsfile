pipeline {
  agent any

  tools {
    
    nodejs 'NodeJS'
  }

  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '25'))
  }

  parameters {
    // <-- Edit these lists when you add new files
    choice(
      name: 'COLLECTION',
      choices: [
        'ACH Processing - Back Office.postman_collection.json',
        'Credit Card Processing - Back Office.postman_collection.json'
      ],
      description: 'Select Postman collection to run'
    )
    choice(
      name: 'ENVIRONMENT',
      choices: [
        'SuitePayments - Visa - Release QA.postman_environment.json',
        'SuitePayments - Visa - UAT - Vishali.postman_environment'
      ],
      description: 'Select Postman environment to use'
    )
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prep Workspace & Reporter') {
      steps {
        bat '''
          if exist "allure-results" rd /s /q "allure-results"
          if exist "allure-report"  rd /s /q "allure-report"
          mkdir "allure-results"

          if not exist package.json (echo {}>package.json)
          call npm config set fund false
          call npm config set audit false
          rem Ensure allure reporter is available to newman (local install in workspace)
          call npm i newman-reporter-allure --no-audit --silent
        '''
      }
    }

    stage('Run Newman Tests') {
      steps {
        script {
          def collectionPath  = "collections/${params.COLLECTION}"
          def environmentPath = "environments/${params.ENVIRONMENT}"

          // Write Allure environment info (for the Environment tab + trend context)
          bat """
            >  "allure-results\\environment.properties" echo BUILD_URL=%BUILD_URL%
            >> "allure-results\\environment.properties" echo JOB_NAME=%JOB_NAME%
            >> "allure-results\\environment.properties" echo BUILD_NUMBER=%BUILD_NUMBER%
            >> "allure-results\\environment.properties" echo NODE_NAME=%NODE_NAME%
            >> "allure-results\\environment.properties" echo COLLECTION=${params.COLLECTION}
            >> "allure-results\\environment.properties" echo ENVIRONMENT=${params.ENVIRONMENT}
          """

          // Run ONLY the selected collection
          bat """
            npx newman run "${collectionPath}" -e "${environmentPath}" -r cli,allure --reporter-allure-export "allure-results"
          """
          // NOTE: If any test fails, newman exits non-zero and this stage fails (good).
        }
      }
    }
  }

  post {
    // Always publish Allure, even if tests failed or the stage errored out
    always {
      allure([
        includeProperties: false,
        jdk: '',
        results: [[path: 'allure-results']]
      ])
      archiveArtifacts artifacts: 'allure-results/**', fingerprint: true, allowEmptyArchive: true
    }
    success {
      echo ' Newman tests passed.'
    }
    failure {
      echo ' Newman tests failed. See the Allure report above.'
    }
  }
}