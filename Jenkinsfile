pipeline {
    agent any

    tools {
        maven "maven3.9.6"
    }

    stages {
        stage("Git Clone") {
            steps {
                git 'https://github.com/kessiey/EcommerceApp.git'
            }
        }

        stage('Build, Test and Package with Maven') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn clean test package'
                }
            }
        }
                
        stage('Scan with Sonarqube') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }

            steps {
                echo "SonarQube Scanning and Analysis..."
                script {
                    dir('EcommerceApp') {
                        def compiledClassesDir = sh(script: 'mvn help:evaluate -Dexpression=project.build.outputDirectory -q -DforceStdout', returnStdout: true).trim()

                        withSonarQubeEnv('sonarqube') {
                            sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=ecommerceapp -Dsonar.java.binaries=${compiledClassesDir}"
                        }
                    }
                }  
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'EcommerceApp', classifier: '', file: '/var/lib/jenkins/workspace/ecommerce-app/EcommerceApp/target/EcommerceApp.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com', nexusUrl: '3.145.216.253:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'ecommerceapp-snapshot', version: '0.0.1-SNAPSHOT'
            }
        }

        stage('Deploy to UAT') {
            steps {
                dir('EcommerceApp') {
                    deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://3.135.209.206:8080/')], contextPath: null, war: 'target/*.war'
                }
            }
        }
    }   
}

