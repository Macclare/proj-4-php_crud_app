pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://16.170.2.219:8081/nexus'
        NEXUS_REPO = 'releases'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
        SONAR_AUTH_TOKEN = 'sonar-token'
        SONAR_URL = 'http://51.21.214.30:9000'
        TAR_FILE_NAME = 'proj-4-php_crud_app.tar.gz'
    }

    tools {
    sonarScanner 'SonarQube Scanner'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "dev", url: 'https://github.com/Macclare/proj-4-php_crud_app.git'
            }
        }

        stage('Check PHP Extensions') {
            steps {
                sh 'php -m | grep mysqli || echo "mysqli extension not found!"'
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
                script {
                    withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIALS_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh """
                        curl -u "$NEXUS_USER:$NEXUS_PASS" --upload-file ${TAR_FILE_NAME} \
                        "${NEXUS_URL}/content/repositories/${NEXUS_REPO}/${TAR_FILE_NAME}"
                        """
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube Scanner'
                    withSonarQubeEnv('sonar-token') { 
                        sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=php-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                        '''
                    }
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
