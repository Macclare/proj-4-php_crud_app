pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://16.171.235.3:8081/nexus'
        NEXUS_REPO = 'php-artifacts'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "dev", url: 'https://github.com/Macclare/proj-4-php_crud_app.git'
            }
        }

        stage('Run PHP Script') {
            steps {
                sh '''
                php -r "echo 'PHP is installed and working!\n';"
                php index.php
                '''
            }
        }

        stage('Package Application') {
            steps {
                sh '''
                tar --exclude=php-crud-app.tar.gz -czvf php-crud-app.tar.gz *
                ls -lh php-crud-app.tar.gz
                '''
            }
        }

        stage('Verify File Before Upload') {
            steps {
                sh 'ls -lh'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                curl -u "admin:admin123" --upload-file php-crud-app.tar.gz \
                "http://16.171.235.3:8081/nexus/repository/php-artifacts/php-crud-app.tar.gz"
                '''
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh 'ansible-playbook -i /etc/ansible/hosts php-playbook.yml'
            }
        }
        
        stage('Manual Promotion to Production') {
            input {
                message "Deploy to Production?"
                ok "Yes, Deploy"
            }
            steps {
                sh 'ansible-playbook -i /etc/ansible/hosts php-playbook.yml'
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline execution failed!"
        }
    }
}
