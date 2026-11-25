pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Prepare workspace') {
            steps {
                dir('test-artifacts') {
                    deleteDir()
                }
            }
        }

        stage('Install dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'npm ci'
                    } else {
                        bat 'npm ci'
                    }
                }
            }
        }

        stage('Run Cypress E2E') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'npm run test:e2e'
                    } else {
                        bat 'npm run test:e2e'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts(
                artifacts: 'test-artifacts/reports/**,test-artifacts/screenshots/**',
                allowEmptyArchive: true
            )
            script {
                if (fileExists('test-artifacts/reports/index.html')) {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'test-artifacts/reports',
                        reportFiles: 'index.html',
                        reportName: 'Cypress HTML Report'
                    ])
                } else {
                    echo 'Cypress HTML report was not created; see the build console output.'
                }
            }
        }
    }
}
