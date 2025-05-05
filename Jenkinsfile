pipeline {
    agent any
    
    environment {
        DOCKER_HOST = 'tcp://192.168.0.7:2376'
    }
    
    stages {
        stage('Checkstyle') {
            steps {
                // Using Gradle for checkstyle based on your existing build.gradle configuration
                sh './gradlew checkstyleMain checkstyleTest'
                
                // Archive the checkstyle results as a job artifact
                archiveArtifacts artifacts: 'build/reports/checkstyle/*.xml', allowEmptyArchive: true
                
                // Optional: publish checkstyle results using the checkstyle plugin
                recordIssues(tools: [checkStyle(pattern: 'build/reports/checkstyle/*.xml')])
            }
        }
        
        stage('Test') {
            steps {
                // Run tests with Gradle
                sh './gradlew test'
            }
            post {
                always {
                    // Publish test results
                    junit '**/build/test-results/test/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                // Build without tests
                sh './gradlew assemble -x test'
            }
        }
        
        stage('Create Docker Image') {
            steps {
                script {
                    // Get the short commit hash
                    def gitCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    
                    // Set the EC2 public IP or DNS - replace this with your actual EC2 public IP or DNS
                    def nexusHost = "34.226.72.65:8081"
                    
                    withCredentials([usernamePassword(credentialsId: 'nexus-docker-credentials', 
                                                    usernameVariable: 'NEXUS_USERNAME', 
                                                    passwordVariable: 'NEXUS_PASSWORD')]) {
                        // Configure Docker to connect to remote Docker host
                        // This assumes you've set up DOCKER_HOST environment variable or Docker context
                        
                        // Login to Nexus Docker registry
                        sh "echo ${NEXUS_PASSWORD} | docker login ${nexusHost} -u ${NEXUS_USERNAME} --password-stdin"
                        
                        // Build the Docker image with the Dockerfile in the repo
                        sh "docker build -t spring-petclinic:${gitCommit} ."
                        
                        // Tag the image for your Nexus repository
                        sh "docker tag spring-petclinic:${gitCommit} ${nexusHost}/repository/mr/spring-petclinic:${gitCommit}"
                        
                        // Push to the "mr" repository
                        sh "docker push ${nexusHost}/repository/mr/spring-petclinic:${gitCommit}"
                        
                        // Logout after push
                        sh "docker logout ${nexusHost}"
                    }
                }
            }
        }
    }
}