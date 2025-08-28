pipeline {
    agent any

    parameters {
        choice(name: 'RUN_MODE', choices: ['single', 'all'], description: 'Choose whether to run one collection or all')
        choice(name: 'ENV', choices: ['SuitePayments - Visa - Release QA.postman_environment.json', 'SuitePayments - Visa - UAT - Vishali.postman_environment.json'], description: 'Select environment file')
        choice(name: 'COLLECTION', choices: ['ACH Processing - Back Office.postman_collection.json', 'Credit Card Processing - Back Office.postman_collection.json'], description: 'Select a single collection (used only if RUN_MODE=single)')
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install -g newman newman-reporter-allure
                '''
            }
        }

        stage('Run Postman Tests') {
            steps {
                script {
                    if (params.RUN_MODE == 'single') {
                        // Run selected collection
                        sh """
                            newman run postman/collections/${params.COLLECTION} \
                            -e postman/environments/${params.ENV} \
                            --reporters cli,allure \
                            --reporter-allure-export results/allure/${params.COLLECTION}
                        """
                    } else {
                        // Run all collections in folder
                        sh """
                            mkdir -p results/allure
                            for file in postman/collections/*.json; do
                                echo "Running collection: $file"
                                name=$(basename "$file" .json)
                                newman run "$file" \
                                -e postman/environments/${params.ENV} \
                                --reporters cli,allure \
                                --reporter-allure-export results/allure/$name
                            done
                        """
                    }
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'results/allure']]
            }
        }
    }
}
