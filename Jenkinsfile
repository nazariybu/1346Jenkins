pipeline {
    agent {
                label 'ubuntu24'
            }

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

    stage('Check Apache Logs') {
            steps {
                script {
                    // Перевірка логів на 4xx/5xx помилки
                    def errors = sh(script: 'grep -E " 4[0-9]{2} | 5[0-9]{2} " /var/log/apache2/error.log | wc -l', returnStdout: true).trim()
                    if (errors != "0") {
                        echo "Знайдено помилки в логах Apache!"
                        // Можна вивести деталі:
                        sh 'grep -E " 4[0-9]{2} | 5[0-9]{2} " /var/log/apache2/error.log'
                        // Якщо потрібно "фейлити" білд:
                        // error("Знайдено помилки в логах Apache!")
                    } else {
                        echo "Помилок 4xx/5xx не знайдено."
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
