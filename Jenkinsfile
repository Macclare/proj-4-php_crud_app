pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://13.48.45.237:8081/nexus'
        NEXUS_REPO = 'php-artifacts'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
        SONAR_AUTH_TOKEN = 'sonar-token'
        SONAR_URL = 'http://http://51.21.255.105:9000'
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
                tar --exclude=proj-4-php_crud_app.tar.gz -czvf proj-4-php_crud_app.tar.gz *
                ls -lh proj-4-php_crud_app.tar.gz
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
                curl -u "admin:admin123" --upload-file proj-4-php_crud_app.tar.gz \
                "http://13.48.45.237:8081/nexus/repository/php-artifacts/proj-4-php_crud_app.tar.gz"
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnvironmentalAnalysis('sonar-id')
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=php-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=${SONAR_URL}
                    -Dsonar.login=${SONAR_AUTH_TOKEN} 
                    '''
                }
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
