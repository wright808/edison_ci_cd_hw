pipeline {
    agent any
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
        stage('JaCoCo Report') {
            steps {
                // Generate JaCoCo report
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn jacoco:report'
                }
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
        stage('Coverage Report') {
            steps {
                // Publish coverage results using the Coverage plugin
                publishCoverage adapters: [jacocoAdapter('target/site/jacoco/jacoco.xml')]
            }
        }
    }
    post {
        success {
            script {
                def jacocoReport = readFile('target/site/jacoco/index.html')
                def cloverReport = readFile('target/site/clover/index.html')
                emailext (
                    subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Build was successful. Check the details at ${env.BUILD_URL}
                        JaCoCo Report: ${jacocoReport}
                        Clover Report: ${cloverReport}
                    """,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                )
            }
        }
        failure {
            script {
                def jacocoReport = readFile('target/site/jacoco/index.html')
                def cloverReport = readFile('target/site/clover/index.html')
                emailext (
                    subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Build failed. Check the details at ${env.BUILD_URL}
                        JaCoCo Report: ${jacocoReport}
                        Clover Report: ${cloverReport}
                    """,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                )
            }
        }
    }
}