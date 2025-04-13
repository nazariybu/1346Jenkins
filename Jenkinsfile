pipeline {
    agent jenkins-agent
    
    environment {
        // Remote server configuration
        REMOTE_HOST = '192.168.56.10'  // IP or hostname
        REMOTE_USER = 'vagrant'       // SSH user
        SSH_KEY = credentials('ssh-private-key')  // SSH key from Jenkins Credentials
        
        // GitHub configuration
        GIT_REPO = 'https://github.com/nazariybu/1346Jenkins.git'
        GIT_BRANCH = 'main'
    }
    
    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
                echo 'Repository successfully cloned'
            }
        }
        
        stage('Install Apache HTTP Server') {
            steps {
                script {
                    sshagent([env.SSH_KEY]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} '
                                # Detect OS and install Apache
                                if [ -f /etc/os-release ]; then
                                    . /etc/os-release
                                    case \$ID in
                                        debian|ubuntu)
                                            sudo apt-get update -y
                                            sudo apt-get install -y apache2
                                            sudo systemctl enable apache2
                                            sudo systemctl start apache2
                                            LOG_FILE="/var/log/apache2/error.log"
                                            ;;
                                        centos|rhel|fedora)
                                            sudo yum install -y httpd
                                            sudo systemctl enable httpd
                                            sudo systemctl start httpd
                                            LOG_FILE="/var/log/httpd/error_log"
                                            ;;
                                        opensuse|sles)
                                            sudo zypper refresh
                                            sudo zypper install -y apache2
                                            sudo systemctl enable apache2
                                            sudo systemctl start apache2
                                            LOG_FILE="/var/log/apache2/error_log"
                                            ;;
                                        *)
                                            echo "Unsupported OS: \$ID"
                                            exit 1
                                            ;;
                                    esac
                                else
                                    echo "Cannot determine OS"
                                    exit 1
                                fi
                                
                                # Create test page
                                echo "<html><body><h1>Apache installed via Jenkins!</h1></body></html>" | sudo tee /var/www/html/index.html
                                echo "LOG_FILE=\$LOG_FILE" > ~/apache_vars
                            '
                        """
                    }
                }
            }
        }
        
        stage('Verify Installation') {
            steps {
                script {
                    sshagent([env.SSH_KEY]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} '
                                source ~/apache_vars
                                # Check service status
                                if command -v systemctl &>/dev/null; then
                                    sudo systemctl status apache2 || sudo systemctl status httpd
                                else
                                    sudo service apache2 status || sudo service httpd status
                                fi
                                
                                # Verify web server response
                                curl -I http://localhost || exit 1
                            '
                        """
                    }
                    echo "Apache HTTP Server successfully installed and running on ${env.REMOTE_HOST}"
                }
            }
        }
        
        stage('Monitor Logs for Errors') {
            steps {
                script {
                    sshagent([env.SSH_KEY]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} '
                                source ~/apache_vars
                                echo "Checking for errors in \$LOG_FILE"
                                
                                # Count 4xx and 5xx errors
                                ERROR_COUNT_4XX=\$(sudo grep -c "HTTP/1.\" [4][0-9][0-9]" \$LOG_FILE || true)
                                ERROR_COUNT_5XX=\$(sudo grep -c "HTTP/1.\" [5][0-9][0-9]" \$LOG_FILE || true)
                                
                                echo "Found \$ERROR_COUNT_4XX 4xx client errors"
                                echo "Found \$ERROR_COUNT_5XX 5xx server errors"
                                
                                if [ \$ERROR_COUNT_4XX -gt 0 ] || [ \$ERROR_COUNT_5XX -gt 0 ]; then
                                    echo "Errors detected in Apache logs"
                                    echo "Last 10 error lines:"
                                    sudo grep "HTTP/1.\" [4-5][0-9][0-9]" \$LOG_FILE | tail -n 10
                                    exit 1
                                fi
                            '
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed"
            // Cleanup if needed
        }
        success {
            echo "Apache deployment and verification successful!"
        }
        failure {
            echo "Pipeline failed - check the logs for details"
            // Send notification if needed
        }
    }
}
