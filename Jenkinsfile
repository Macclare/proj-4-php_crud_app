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
                tar -czvf php-crud-app.tar.gz .
                '''
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "http://16.171.235.3:8081/nexus",
                    groupId: 'com.macclare.php',
                    version: '1.0.0',
                    repository: "php-artifacts",
                    credentialsId: "nexus-credentials",
                    artifacts: [
                        [artifactId: 'php-crud-app', classifier: '', file: 'php-crud-app.tar.gz', type: 'tar.gz']
                    ]
                )
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
