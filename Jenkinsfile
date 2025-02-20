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
                                if [ ! -d "$REMOTE_DIR" ]; then
                                    echo "Cloning repository to $REMOTE_DIR"
                                    git clone https://github.com/Sangameshwar-652/python-flask-app.git $REMOTE_DIR
                                else
                                    echo "Directory $REMOTE_DIR exists, pulling latest changes"
                                    cd $REMOTE_DIR
                                    git pull origin main
                                fi

                                # Install dependencies
                                cd $REMOTE_DIR
                                python3 --version
                                python3 -m pip install --upgrade pip
                                python3 -m pip install -r requirements.txt



                                # Stop the Flask app if it's already running
                                pkill -f 'flask run' || true

                                # Run the Flask app
                                nohup flask run --host=0.0.0.0 --port=5000 &

                            EOF
                        """
                    }
                }
            }
        }

    }
 }

