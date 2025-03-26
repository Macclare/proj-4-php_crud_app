pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "dev", url: 'https://github.com/Macclare/proj-4-php_crud_app.git'
            }
        }

        stage('Install PHP and MySQL') {
            steps {
                sh '''
                sudo apt-get update
                sudo apt-get install -y php php-mysqli mysql-client
                '''
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
