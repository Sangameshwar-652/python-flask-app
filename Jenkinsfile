pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        APP_NAME = 'flask-app'
        EC2_USER = 'ec2-user'
        EC2_IP = '172.31.84.177'
        REMOTE_DIR = '/home/ec2-user/your-flask-app'
        DOCKER_USERNAME = 'ktsangameshwar652'  // Docker registry credentials (optional)
        DOCKER_PASSWORD =  'kts6628@@@' // Docker registry credentials (optional)

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
            // Use SSH credentials for EC2 connection
            withCredentials([sshUserPrivateKey(credentialsId: 'ssh', keyFileVariable: 'SSH_KEY')]) {
                sh """
                    ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$EC2_USER@$EC2_IP <<'EOF'
                        # Navigate to the deployment directory
                        cd $REMOTE_DIR || mkdir -p $REMOTE_DIR && cd $REMOTE_DIR

                        # Pull the latest code and update the app
                        git pull origin main

                        # Install dependencies on EC2 (if not installed)
                        python3 -m pip install -r requirements.txt

                        # Start the Flask app (ensure it's running in the background)
                        nohup python3 $FLASK_APP &> flask_app.log &
                    EOF
                """
            }
        }
    }
}
    }
 }

