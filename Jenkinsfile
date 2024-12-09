pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Клонирование репозитория
                checkout scm
            }
        }
        stage('Build') {
            steps {
                // Команда сборки (например, Maven)
                sh './mvnw clean install'
            }
        }
        stage('Test') {
            steps {
                // Команда тестирования
                sh './mvnw test'
            }
        }
    }
    post {
        always {
            // Архивация артефактов
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
        failure {
            // Уведомление об ошибке
            echo 'Сборка завершилась с ошибкой'
        }
    }
}
