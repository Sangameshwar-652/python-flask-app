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
                    // Use SSH credentials for EC2 connection
                    withCredentials([sshUserPrivateKey(credentialsId: 'dev', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$EC2_USER@$EC2_IP <<'EOF'
                                # Check if container is running and stop if necessary
                                if [ \$(docker ps -q --filter "name=$APP_NAME") ]; then
                                    docker stop \$(docker ps -q --filter "name=$APP_NAME")
                                fi

                                # Check if container exists and remove if necessary
                                if [ \$(docker ps -a -q --filter "name=$APP_NAME") ]; then
                                    docker rm \$(docker ps -a -q --filter "name=$APP_NAME")
                                fi

                                # Pull or build the Docker image on the EC2 instance
                                # If not using a registry, you could copy the image from Jenkins to EC2
                                docker load -i /tmp/$APP_NAME.tar || true

                                # If the image is not available, build it on EC2
                                if ! docker images -q $APP_NAME; then
                                    cd $REMOTE_DIR
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

