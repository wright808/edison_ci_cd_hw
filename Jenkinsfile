pipeline {
    agent any

    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
        stage('Clean') {
            steps {
                // Clean the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clean'
                }
            }
        }
        stage('Build') {
            steps {
                // Build the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn compile'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests using Maven with JaCoCo agent
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn test'
                }
            }
        }
        stage('Package and Install') {
            steps {
                // Package the application
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn install'
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
        stage('Record JaCoCo Coverage') {
            steps {
                // Record JaCoCo coverage using Coverage plugin
                recordCoverage tools: [[parser: 'JACOCO']],
                    id: 'jacoco', name: 'JaCoCo Coverage',
                    sourceCodeRetention: 'EVERY_BUILD',
                    qualityGates: [
                        [threshold: 25.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                        [threshold: 50.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]
                    ]
            }
        }
    }
    post {
        success {
            script {
                def coverageReportPath = 'target/site/jacoco/index.html'
                if (fileExists(coverageReportPath)) {
                    emailext (
                        subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build was successful. Check the details at ${env.BUILD_URL}
                            JaCoCo Coverage Report: ${env.BUILD_URL}/jacoco
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                } else {
                    emailext (
                        subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build was successful. Check the details at ${env.BUILD_URL}
                            JaCoCo Coverage Report: Not generated.
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                }
            }
        }
        failure {
            script {
                def coverageReportPath = 'target/site/jacoco/index.html'
                if (fileExists(coverageReportPath)) {
                    emailext (
                        subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed. Check the details at ${env.BUILD_URL}
                            JaCoCo Coverage Report: ${env.BUILD_URL}/jacoco
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                } else {
                    emailext (
                        subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed. Check the details at ${env.BUILD_URL}
                            JaCoCo Coverage Report: Not generated.
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                }
            }
        }
    }
}
