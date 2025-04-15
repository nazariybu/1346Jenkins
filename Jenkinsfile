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

        stage('Check Apache Logs') {
            steps {
                script {
                    // Шукаємо помилки 4xx та 5xx і рахуємо їх кількість
                    def errorCount = sh(
                        script: 'grep -E " 4[0-9]{2} | 5[0-9]{2} " /var/log/apache2/access.log | wc -l',
                        returnStdout: true
                    ).trim()

                    if (errorCount != "0") {
                        // Виводимо деталі помилок (опціонально)
                        def errorDetails = sh(
                            script: 'grep -E " 4[0-9]{2} | 5[0-9]{2} " /var/log/apache2/access.log | head -n 10',
                            returnStdout: true
                        ).trim()
                        
                        // Повідомлення з кількістю помилок
                        echo "Знайдено ${errorCount} помилок в логах Apache! Приклад перших 10:"
                        echo "${errorDetails}"
                        
                        // Можна "фейлити" білд, якщо є помилки
                        // error("Є критичні помилки в логах Apache!")
                    } else {
                        echo "Помилок 4xx/5xx не знайдено."
                    }
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
