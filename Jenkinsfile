pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'JDK'
    }

    parameters {
        choice(
            name: 'CHOICE',
            choices: ['prod', 'dev', 'uat'],
            description: 'Choose any environment'
        )
    }

    stages {

        stage('Checkout') {
            when {
                expression { params.CHOICE == 'prod' }
            }

            steps {
                git 'https://github.com/OT-MyGurukulam/spring3hibernate.git'
            }
        }

        stage('GitLeaks Scan') {
            when {
                expression { params.CHOICE == 'prod' }
            }

            steps {
                sh '''
                    gitleaks detect \
                    --source . \
                    --report-format json \
                    --report-path gitleaks-report.json \
                    || true
                '''
            }
        }

        stage('Approval') {
            when {
                expression { params.CHOICE == 'prod' }
            }

            steps {
                input message: 'Proceed with Maven Build?',
                      ok: 'Start Maven Build'
            }
        }

        stage('Maven Build') {
            when {
                expression { params.CHOICE == 'prod' }
            }

            parallel {
                stage('Maven Compile') {
                    steps {
                        echo "Compiling project through Maven"
                        sh 'mvn compile || true'
                    }
                }

                stage('Maven Test') {
                    steps {
                        echo "Running tests through Maven"
                        sh 'mvn test || true'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
        }

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed. Check console output."
        }
    }
}
