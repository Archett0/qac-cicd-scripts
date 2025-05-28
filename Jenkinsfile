pipeline {
    agent any

    tools {
        jdk "jdk21"
        maven "maven3"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'cicd', changelog: false, poll: false, url: 'https://github.com/Archett0/smr-backend.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube-main') {
                    sh 'mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
                }
            }
        }
//         stage('OWASP scan') {
//             steps {
//                 dependencyCheck additionalArguments: ' --scan ./', nvdCredentialsId: '9274bf55-b151-4a91-ae5b-53d257d790ba', odcInstallation: 'DPC'
//                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
//             }
//         }
        stage('Maven build') {
            steps {
                sh "mvn clean install -DskipTests=true"
            }
        }
        stage('Build and push images') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh "gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS"
                    sh "gcloud config set project mimetic-codex-460815-g8"
                    sh "gcloud auth configure-docker us-central1-docker.pkg.dev"
                    dir('CloudGateway') {
                        sh "docker build -t us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/cloudgateway:latest ."
                        sh "docker push us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/cloudgateway:latest"
                    }
                    dir('ListingService') {
                        sh "docker build -t us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/listingservice:latest ."
                        sh "docker push us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/listingservice:latest"
                    }
                    dir('UserService') {
                        sh "docker build -t us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/userservice:latest ."
                        sh "docker push us-central1-docker.pkg.dev/mimetic-codex-460815-g8/smr-repo/userservice:latest"
                    }
                }
            }
        }
//         stage('Trigger CD pipeline') {
//             steps {
//                 build wait: false, propagate: false, job: 'smr-cd-pipeline', waitForStart: true
//             }
//         }
    }
    post {
        always {
            cleanWs()
        }
    }
}
