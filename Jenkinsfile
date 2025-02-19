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

    }
}
