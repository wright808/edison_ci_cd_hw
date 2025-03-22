pipeline {
    agent any

    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
        stage('Build') {
            steps {
                // Build the project using Maven
                
                bat 'mvn clean install'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests using Maven
                
                bat 'mvn test'
                }
            }
        }
        stage('Package') {
            steps {
                // Package the application
                
                bat 'mvn package'
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
            echo 'Build, Test, and Package stages completed successfully!'
        }
        failure {
            echo 'Build, Test, or Package stage failed.'
        }
    }
}