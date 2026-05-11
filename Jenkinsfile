pipeline {
    agent any
    tools {
        maven 'Maven3' // Must match the name in Jenkins Global Tool Config 
    }
    environment {
        // Force the environment to use Java 21
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Git Checkout') {
            steps { checkout scm }
        }
         stage('Environment Check') {
            steps {
                // This will prove to you in the logs if it's actually using 21
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        stage('Maven Build & Unit Test') {
            steps {
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
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'OWASP-DC'
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
 
