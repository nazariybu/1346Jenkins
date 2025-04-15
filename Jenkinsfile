pipeline {
    agent {lable 'ubuntu24'}

    stages {
        stage('Install Apache2') {
            steps {
                script {
                    // Оновлення пакетів та встановлення Apache2
                    sh 'sudo apt-get update -y'
                    sh 'sudo apt-get install apache2 -y'
                }
            }
        }

        stage('Start Apache2') {
            steps {
                script {
                    // Запуск Apache2 та перевірка статусу
                    sh 'sudo systemctl start apache2'
                    sh 'sudo systemctl enable apache2'
                    sh 'sudo systemctl status apache2 || true'
                }
            }
        }

        stage('Verify Installation') {
            steps {
                script {
                    // Проста перевірка, чи Apache2 працює
                    sh 'curl -I http://localhost || true'
                }
            }
        }
    }

    post {
        always {
            echo 'Apache2 installation process completed'
        }
        success {
            echo 'Apache2 was successfully installed and started!'
        }
        failure {
            echo 'Apache2 installation failed!'
        }
    }
}
