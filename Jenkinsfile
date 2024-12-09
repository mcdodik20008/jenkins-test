pipeline {
    agent any

    tools {
        maven 'maaaven' // Убедитесь, что Maven3 настроен в Global Tool Configuration
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
}
