pipeline {
    agent any
    tools {
        maven 'Maven3' // Must match the name in Jenkins Global Tool Config 
        jdk 'jdk21'
    }
    stages {
        stage('Git Checkout') {
            steps { checkout scm }
        }
        stage('Maven Build & Unit Test') {
            steps { 
                sh 'java -version'
                sh 'mvn clean package' 
            }
        }
        stage('SonarQube Quality Check') {
            steps {
                withSonarQubeEnv('SonarQube-Server') { 
                    sh 'mvn sonar:sonar' 
                }
            }
        }
        stage('Security Scan (Dependency-Check)') {
            steps {
                dependencyCheck odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t my-devops-app:latest .'
                // Optional: add push steps here if you have credentials set up
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL my-devops-app:latest'
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker rm -f my-running-app || true'
                sh 'docker run -d --name my-running-app -p 8081:8080 my-devops-app:latest'
            }
        }
    }
}
 
