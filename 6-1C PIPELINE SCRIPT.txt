pipeline {
    agent any
    
    triggers {
        pollSCM('* * * * *')  // Poll every minute
    }

    environment {
        DIRECTORY_PATH = 'C:\\Users\\lenovo\\Documents\\Deakin Units S334\\SIT_223_PPIT\\Github'
        TESTING_ENVIRONMENT = 'testing'
        PRODUCTION_ENVIRONMENT = 'V'
        RECP = 'hagishichirukicha@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jerryd10/jenkins.git/'
            }
        }

        stage('Build'){
            steps{
                echo "Building (Tool: Maven)"
                echo "Maven: Retrieving source code from ${env.DIRECTORY_PATH}"
                echo "Maven: Compiling..."
                echo "Maven: Packaging into JAR file..."
            }
        }
        stage('Unit and Integration Tests') {
            steps {
                echo "Unit testing (Tool: JUnit 5)"
                echo "JUnit 5: unit testing..."
                echo "Integration testing (Tool: JUnit 5)"
                echo "JUnit 5: integration testing..."
            }
            post{
                success{
                    mail to: "${env.RECP}",
                        subject: 'Unit and Integration Tests: SUCCESS',
                        body: 'Unit and integration tests were successful.'
                }
                failure{
                    mail to: "${env.RECP}",
                        subject: 'Unit and Integration Tests: FAILURE',
                        body: 'Unit and integration tests failed.'
                }
            }
        }
        stage('Code Analysis') {
            steps {
                echo "Analysing the quality of the code (Tool: SonarQube)"
                echo "SonarQube: analyzing source code..."
            }
        }
        stage('Security Scan') {
            steps {
                echo "Performing security scan of source code (Tool: Fortify)"
                echo "Fortify: checking for vulnerabilities..."
            }
            post{
                success{
                    mail to: "${env.RECP}",
                        subject: 'Security Scan: SUCCESS',
                        body: 'Security scans revealed no issues.'
                }
                failure{
                    mail to: "${env.RECP}",
                        subject: 'Security Scan: FAILURE',
                        body: 'Security scans revealed one or more issues. Please log in and resolve them.'
                }
            }
        }
        stage('Deploy to Staging Server') {
            steps {
                echo "Deploying to staging server: AWS EC2 (Tool: Terraform)"
                echo "Terraform: deploying to staging server: AWS EC2..."
            }
        }
        stage('Integration Tests on Staging Server') {
            steps {
                echo "Testing on staging server (Tool: Cypress)"
                echo "Cypress: testing..."
            }
            post{
                success{
                    mail to: "${env.RECP}",
                        subject: 'Integration Tests on Staging Server: SUCCESS',
                        body: 'Integration tests on staging server were successful.'
                }
                failure{
                    mail to: "${env.RECP}",
                        subject: 'Integration Tests on Staging Server: FAILURE',
                        body: 'Integration tests failed on staging server.'
                }
            }
        }
        stage('Deploy to Production Server') {
            steps {
                echo "Deploying the code to the production server: AWS EC2 (Tool: Terraform)"
                echo "Terraform: deploying to production server: AWS EC2..."
            }
        }
        
        stage('END') {
            steps{
                echo "ENDING..."
            }
            post{
                success{
                    emailext(
                        to: "${env.RECP}",
                        subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!',
                        body: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS: Check console output at $BUILD_URL to view the results.',
                        attachLog: true
                    )
                }
                
                failure{
                    emailext(
                        to: "${env.RECP}",
                        subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!',
                        body: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS: Check console output at $BUILD_URL to view the results.',
                        attachLog: true
                    )
                }
                
            }
            
        }
    }
}