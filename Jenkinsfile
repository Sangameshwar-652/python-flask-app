pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        APP_NAME = 'flask-app'
        EC2_USER = 'ec2-user'
        EC2_IP = '172.31.84.177'
        REMOTE_DIR = '/home/ec2-user/your-flask-app'

    }

    stages {
        stage ("Clean") {
            steps {
                cleanWs()
            }
        }
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Sangameshwar-652/python-flask-app.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'python3 --version'
                    sh 'python3 -m pip install --upgrade pip'
                    sh 'python3 -m pip install -r requirements.txt'
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

        stage('Deploy') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            echo "Deploying application to EC2..."

                            # SSH into the EC2 instance and execute deployment commands
                            ssh -i $SSH_KEY $EC2_USER@$EC2_IP << 'EOF'
                                cd $REMOTE_DIR
                                git pull origin main  # Pull the latest code
                                source /home/$EC2_USER/.bash_profile  # Load virtual environment or settings
                                python3 -m pip install -r requirements.txt  # Install dependencies on the remote server
                                sudo systemctl restart flask-app.service || echo "Flask app service not found!"  # Restart the app
                            EOF

                            echo "Deployment completed successfully!"
                        '''
                    }
                }
            }
     }
}
}
