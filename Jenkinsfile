pipeline {
    agent any

    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
        stage('Set Environment Variable') {
            steps {
                script {
                    env.IS_PULL_REQUEST = 'false'
                    if (env.BRANCH_NAME != null)
                        if (env.BRANCH_NAME.startsWith('PR-')) {
                            env.IS_PULL_REQUEST = 'true'
                        }
                }
            }
        }
        stage('Build') {
            steps {
                // Build the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clean install -Dmaven.test.skip'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clover:setup'
                    bat 'mvn clean test'
                }
            }
        }
        stage('Package') {
            steps {
                // Package the application
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn package -Dmaven.test.skip'
                }
            }
        }
        stage('Publish Test Results') {
            steps {
                // Publish JUnit test results
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Clover Report') {
            steps {
                // Generate Clover report
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clover:clover'
                }
            }
        }
        stage('Report') {
            steps {
                clover(cloverReportDir: 'target/site', cloverReportFileName: 'clover.xml',
                // optional, default is: method=70, conditional=80, statement=80
                healthyTarget: [methodCoverage: 70, conditionalCoverage: 80, statementCoverage: 80],
                // optional, default is none
                unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50],
                // optional, default is none
                failingTarget: [methodCoverage: 0, conditionalCoverage: 0, statementCoverage: 0]
                )
            }
    }
    post {
        success {
            script {
                def cloverReportPath = 'target/site/clover/index.html'
                if (fileExists(cloverReportPath)) {
                    def cloverReport = readFile(cloverReportPath)
                    emailext (
                        subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build was successful. Check the details at ${env.BUILD_URL}
                            Clover Report: ${cloverReport}
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                } else {
                    emailext (
                        subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build was successful. Check the details at ${env.BUILD_URL}
                            Clover Report: Not generated.
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                }
            }
        }
        failure {
            script {
                def cloverReportPath = 'target/site/clover/index.html'
                if (fileExists(cloverReportPath)) {
                    def cloverReport = readFile(cloverReportPath)
                    emailext (
                        subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed. Check the details at ${env.BUILD_URL}
                            Clover Report: ${cloverReport}
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                } else {
                    emailext (
                        subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed. Check the details at ${env.BUILD_URL}
                            Clover Report: Not generated.
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                }
            }
        }
    }
}