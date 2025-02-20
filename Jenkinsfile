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
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image from the current directory
                    sh 'docker build -t $APP_NAME .'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Use SSH credentials for EC2 connection
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$EC2_USER@$EC2_IP <<EOF
                                                               # Navigate to the home directory
                                cd /home/ec2-user

                                # If the app directory doesn't exist, clone the repo
                                if [ ! -d "$REMOTE_DIR" ]; then
                                    echo "Cloning the repository..."
                                    git clone https://github.com/Sangameshwar-652/python-flask-app.git $REMOTE_DIR
                                fi

                                # Navigate to the app directory
                                cd $REMOTE_DIR

                                # Pull the latest changes from the repository
                                git pull origin main

                                # Install dependencies (this is assuming you're using pip3)
                                python3 --version
                                python3 -m pip install --upgrade pip
                                pip3 install -r requirements.txt

                                # Stop any running Flask app (if necessary)
                                pkill -f 'flask run' || true

                                # Run the Flask app in the background
                                nohup flask run --host=0.0.0.0 --port=5000 &


                            EOF
                        """
                    }
                }
            }
        }

    }
 }

