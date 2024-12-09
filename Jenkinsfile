pipeline {
    agent any

    tools {
        maven 'maaaven' // Имя Maven из Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                // Публикуем артефакты в Nexus
                sh '''
                mvn deploy -DaltDeploymentRepository=nexus::default::http://localhost:8081/repository/test-repo/
                '''
            }
        }
    }
}
