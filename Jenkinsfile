pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment{
        
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git 'https://github.com/jaiswaladi246/secretsanta-generator.git'
            }
        }
        stage('Complie') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=santa \
                    -Dsonar.projectKey=santa -Dsonar.java.binaries=. '''
    
                }
            }
        }
        
        stage('Build Application') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Owasp Scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan . ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        sh "docker build -t santa:latest ."
                    }
                }
            }
        }
        stage('Tag and Push Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        sh "docker tag santa:latest varunkumarmc/santa:latest"
                        sh "docker push varunkumarmc/santa:latest"
                    }
                }
            }
        }
        stage('Deploy Application') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        sh "docker run -d -p 8081:8080 varunkumarmc/santa:latest"
                    }
                }
            }
        }
    }
    post {
            always {
                emailext (
                    subject: "Pipeline Status: ${BUILD_NUMBER}",
                    body: '''<html>
                                <body>
                                    <p>Build Status: ${BUILD_STATUS}</p>
                                    <p>Build Number: ${BUILD_NUMBER}</p>
                                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                                </body>
                            </html>''',
                    to: 'jaiswaladi246@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html'
                )
     }
    }
}
