pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        APP_NAME = 'flask-app'
        EC2_USER = 'Production'
        EC2_IP = '172.31.95.210'
        REMOTE_DIR = '/home/ec2-user/your-flask-app'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Sangameshwar-652/python-flask-app.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                sh '''
                export PATH=$PATH:/var/lib/jenkins/.local/bin
                pytest
                '''        
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                   sh '''#!/bin/bash
                    set -e
                    npm install -g pm2
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP << EOF
                    cd $REMOTE_DIR
                    git pull origin main
                    pip install -r requirements.txt
                    pm2 restart flask-app || pm2 start app.py --name flask-app
                    EOF
                }
            }
    }
}
