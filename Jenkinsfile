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
                git branch: 'sec-sdlc', changelog: false, poll: false, url: 'https://github.com/Archett0/qac-sec.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube-main') {
                    bat 'mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
                }
            }
        }
        stage('OWASP scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', nvdCredentialsId: '2e7f308f-5310-4393-b4e9-fb2bd91a3122', odcInstallation: 'DPC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Maven build') {
            steps {
                echo 'Only a test build'
                bat "mvn clean install -DskipTests=true"
            }
        }
        stage('Build images') {
            steps {
                bat 'docker build -t qac-sec-api-gateway gateway/'
                bat 'docker build -t qac-sec-qna QnA/'
                bat 'docker build -t qac-sec-event event/'
                bat 'docker build -t qac-sec-search search/'
                bat 'docker build -t qac-sec-user user/'
                bat 'docker build -t qac-sec-vote vote/'
            }
        }
        stage('Deploy') {
            steps {
                bat 'docker run -d --name gateway --network qac_default --label project=qac-sec-services -p 8080:8080 qac-sec-api-gateway'
                bat 'docker run -d --name qna --network qac_default --label project=qac-sec-services qac-sec-qna'
                bat 'docker run -d --name event --network qac_default --label project=qac-sec-services qac-sec-event'
                bat 'docker run -d --name search --network qac_default --label project=qac-sec-services qac-sec-search'
                bat 'docker run -d --name user --network qac_default --label project=qac-sec-services qac-sec-user'
                bat 'docker run -d --name vote --network qac_default --label project=qac-sec-services qac-sec-vote'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
