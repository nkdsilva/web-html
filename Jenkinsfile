pipeline {

    agent any

    environment {
        S3_BUCKET = 'artifact-index-s3'
        APP_SERVER_IP = '15.134.33.63'
        REMOTE_USER = 'ubuntu'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'dev', url: 'https://github.com/nkdsilva/web-html.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building..."'	
                sh 'aws s3 cp index.html s3://$S3_BUCKET/index.html'
                sh 'echo "End building..."'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['apache-ssh-key']) {
                  sh 'scp index.html ubuntu@$APP_SERVER_IP:/var/www/html/index.html'
                  sh 'ssh ubuntu@APP_SERVER_IP "sudo systemctl restart apache2"'
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    def status = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://$APP_SERVER_IP", returnStdout: true).trim()
                    echo "Apache returned status code: ${status}"
                }
            }
        }

    }

    post {
        success {
            echo 'Build completed successfully!'
        }

        failure {
            echo 'Build failed!'
        }
    }
}