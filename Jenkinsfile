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
            // Create a temporary shell script for deployment
            writeFile file: 'deploy_script.sh', text: """
                #!/bin/bash
                # Ensure Python 3 is installed
                if ! command -v python3 &> /dev/null
                then
                    echo "Python 3 could not be found, installing..."
                    sudo yum install -y python3
                fi

                # Ensure pip is installed and upgraded
                if ! command -v pip3 &> /dev/null
                then
                    echo "pip3 could not be found, installing..."
                    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    sudo python3 get-pip.py
                fi

                # Navigate to the deployment directory
                cd $REMOTE_DIR || mkdir -p $REMOTE_DIR && cd $REMOTE_DIR

                # Pull the latest code and update the app
                git pull origin main

                # Install dependencies on EC2 (if not installed)
                python3 -m pip install --upgrade pip
                python3 -m pip install -r requirements.txt

                # Start the Flask app (ensure it's running in the background)
                nohup python3 $FLASK_APP &> flask_app.log &
            """
            
            // Use SSH credentials for EC2 connection
            withCredentials([sshUserPrivateKey(credentialsId: 'ssh-', keyFileVariable: 'SSH_KEY')]) {
                sh """
                    # Copy the deploy script to EC2
                    scp -i \$SSH_KEY deploy_script.sh \$EC2_USER@$EC2_IP:/home/\$EC2_USER/
                    
                    # SSH into EC2 and run the deploy script
                    ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$EC2_USER@$EC2_IP 'bash /home/\$EC2_USER/deploy_script.sh'
                """
            }
        }
    }
}

    }
}

