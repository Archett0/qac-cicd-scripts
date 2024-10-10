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
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Seven0730/qac.git'
            }
        }

        stage('Maven build') {
            steps {
                echo 'Only a test build'
            }
        }
    }
}
