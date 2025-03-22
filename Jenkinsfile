pipeline {
    agent any

    tools {
        maven 'maven-3.9.9' // Ensure Maven is available
    }

    stages {
        stage('Sanity')
        {
            bat 'dir'
        }
        stage('Build') {
            steps {
                // Build the project using Maven
                withMaven(maven: 'maven-3.9.9') {
                    bat 'mvn clean install'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests using Maven
                withMaven(maven: 'maven-3.9.6') {
                    bat 'mvn test'
                }
            }
        }
        stage('Package') {
            steps {
                // Package the application
                withMaven(maven: 'maven-3.9.6') {
                    bat 'mvn package'
                }
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