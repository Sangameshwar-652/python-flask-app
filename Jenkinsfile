pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        APP_NAME = 'flask-app'
        EC2_USER = 'ec2-user'
        EC2_IP = '54.205.142.1'
        REMOTE_DIR = '/home/ec2-user/your-flask-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Sangameshwar-652/python-flask-app.git', branch: 'main'
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
                    // Deploy to EC2
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "cd ${REMOTE_DIR} && git pull origin main && source /home/${EC2_USER}/.virtualenvs/${APP_NAME}/bin/activate && pip install -r requirements.txt && sudo systemctl restart ${APP_NAME}"
                    '''
                }
            }
        }
    }
}
