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
                            # Deploy to EC2 instance
                            echo "Starting deployment..."

                            # SSH into EC2 instance and pull the latest code
                            ssh -i $SSH_KEY $EC2_USER@$EC2_IP << 'EOF'
                                cd $REMOTE_DIR
                                git pull origin main
                                source /home/$EC2_USER/.bash_profile  # Activate virtualenv or required env
                                pip install -r requirements.txt
                                # Restart the Flask app
                                sudo systemctl restart flask-app.service
                            EOF

                            echo "Deployment complete!"
                        '''
                    }
                }
            }
     }
}
}
