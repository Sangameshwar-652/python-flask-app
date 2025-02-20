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

                                # Check if any container with the name exists and stop/remove it
                                CONTAINER_ID=\$(docker ps -q --filter "name=$APP_NAME")
                                if [ -n "\$CONTAINER_ID" ]; then
                                    docker stop \$CONTAINER_ID
                                    docker rm \$CONTAINER_ID
                                fi

                                # Check if the image exists locally, if not, build it
                                if ! docker image inspect $APP_NAME > /dev/null 2>&1; then
                                    echo "Image $APP_NAME not found locally. Building the image..."
                                    docker build -t $APP_NAME .
                                fi

                                # Run the Flask app in a Docker container
                                docker run -d --name $APP_NAME -p 80:5000 $APP_NAME

                            EOF
                        """
                    }
                }
            }
        }

    }
 }

