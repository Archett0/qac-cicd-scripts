pipeline {
    agent any

    tools {
        jdk "jdk21"
        maven "maven3"
    }

    stages {
        stage('Git checkout') {
            steps {
                // Get code from GitHub repository
                git branch: 'develop', changelog: false, poll: false, url: 'https://github.com/Seven0730/qac.git'
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube-main') {
                    sh 'mvn clean package org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
                }
            }
        }

        stage('OWASP scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DPC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Maven build') {
            steps {
                echo 'Only a test build'
                sh "mvn clean install -DskipTests=true"
            }
        }
    }
}
