pipeline {
    agent any

    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
        stage('Clean') {
            steps {
                // Build the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clean'
                }
            }
        }
        stage('Build') {
            steps {
                // Build the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn compile -Dmaven.test.skip'
                }
            }
        }
        stage('Clover Setup') {
            steps {
                // Setup Clover instrumentation
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clean clover:setup'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn test'
                }
            }
        }
        stage('Package and Install') {
            steps {
                // Package the application
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn install -Dmaven.test.skip'
                }
            }
        }
        stage('Static Analysis') {
            steps {
                // Run Checkstyle static analysis
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn checkstyle:checkstyle'
                }
            }
        }
        stage('Publish Test Results') {
            steps {
                // Publish JUnit test results
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Publish Checkstyle Results') {
            steps {
                // Publish Checkstyle results using Warnings Next Generation Plugin
                recordIssues tools: [checkStyle(pattern: '**/target/checkstyle-result.xml')]
            }
        }
        stage('Clover Report') {
            steps {
                // Generate Clover report
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clover:aggregate clover:clover'
                }
            }
        }
        stage('Publish Checks') {
            steps {
                publishChecks name: 'example', title: 'Pipeline Check', summary: 'check through pipeline',
                    text: 'you can publish checks in pipeline script',
                    detailsURL: 'https://github.com/jenkinsci/checks-api-plugin#pipeline-usage',
                    actions: [[label: 'an-user-request-action', description: 'actions allow users to request pre-defined behaviours', identifier: 'example-action']]
            }
        }
    }
    post {
        success {
            script {
                def cloverReportPath = 'target/site/clover/index.html'
                if (fileExists(cloverReportPath)) {
                    emailext (
                        subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build was successful. Check the details at ${env.BUILD_URL}
                            Clover Report: ${env.BUILD_URL}target/site/clover/index.html
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
                    emailext (
                        subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed. Check the details at ${env.BUILD_URL}
                            Clover Report: ${env.BUILD_URL}target/site/clover/index.html
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
