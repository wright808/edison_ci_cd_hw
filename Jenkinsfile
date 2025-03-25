pipeline {
    agent any
    triggers {
        githubPush()
    }
    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
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
    }

    post {
        success {
            emailext (
                subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build was successful. Check the details at ${env.BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed. Check the details at ${env.BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    }
}