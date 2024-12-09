pipeline {
    agent any

    tools {
        maven 'maaven'
    }

    stages {
        stage('Check Version') {
            steps {
                sh 'mvn -v'
            }
        }
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
