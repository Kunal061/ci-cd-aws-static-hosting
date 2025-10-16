pipeline {
    agent {
        label 'aws-ec2-agent'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                git branch: 'main', url: 'https://github.com/<YOUR_USERNAME>/ci-cd-aws-static-hosting.git'
            }
        }
        
        stage('Deploy to Apache') {
            steps {
                echo 'Deploying files to Apache web server...'
                sh '''
                    # Copy HTML and CSS files to Apache webroot
                    sudo cp -r index.html /var/www/html/
                    sudo cp -r styles.css /var/www/html/
                    
                    # Set proper permissions
                    sudo chown -R www-data:www-data /var/www/html/
                    sudo chmod -R 755 /var/www/html/
                    
                    # Restart Apache to apply changes
                    sudo systemctl restart apache2
                    
                    echo "Deployment completed successfully!"
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh '''
                    # Check if Apache is running
                    sudo systemctl status apache2 --no-pager
                    
                    # List deployed files
                    ls -la /var/www/html/
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully! Website is live.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
