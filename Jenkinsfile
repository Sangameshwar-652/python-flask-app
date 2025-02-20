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
                    // Log into Docker registry (if required)
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''

                    // Optional: Push the image to Docker registry
                    // sh 'docker push $DOCKER_USERNAME/$APP_NAME'

                    // Use SSH credentials for EC2 connection
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$EC2_USER@$EC2_IP <<EOF
                                # Pull the Docker image from the registry (if it's pushed)
                                docker pull $APP_NAME

                                # Stop and remove any existing container with the same name
                                docker stop \$(docker ps -q --filter "name=$APP_NAME") || true
                                docker rm \$(docker ps -a -q --filter "name=$APP_NAME") || true

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

