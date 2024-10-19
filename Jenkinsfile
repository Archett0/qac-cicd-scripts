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
                    sh 'mvn clean package -DskipTests=true org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
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
        stage('Auth to AWS ECR') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'e046f63b-a53a-4c53-bbc5-cbcd1c97c379', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/v0c3x7n3"
                }
            }
        }
        stage('Build and push images') {
            steps {
                dir('gateway') {
                    sh "docker build -t qnac/gateway ."
                    sh "docker tag qnac/gateway:latest public.ecr.aws/v0c3x7n3/qnac/gateway:latest"
                    sh "docker push public.ecr.aws/v0c3x7n3/qnac/gateway:latest"
                }
                dir('QnA') {
                    sh "docker build -t qnac/qna-service ."
                    sh "docker tag qnac/qna-service:latest public.ecr.aws/v0c3x7n3/qnac/qna-service:latest"
                    sh "docker push public.ecr.aws/v0c3x7n3/qnac/qna-service:latest"
                }
//                 dir('event') {
//                     sh ""
//                     sh ""
//                     sh ""
//                 }
//                 dir('user') {
//                     sh ""
//                     sh ""
//                     sh ""
//                 }
//                 dir('vote') {
//                     sh ""
//                     sh ""
//                     sh ""
//                 }
//                 dir('search') {
//                     sh ""
//                     sh ""
//                     sh ""
//                 }
            }
        }
        stage('Trigger CD pipeline') {
            steps {
                build wait: false, propagate: false, job: 'qac-cd-pipeline', waitForStart: true
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
