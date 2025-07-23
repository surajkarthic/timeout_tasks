pipeline {
    agent any
    environment {
        VENV_DIR = 'venv'
        AWS_DEFAULT_REGION = 'ap-south-1'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Setup Environment') {
            steps {
                sh '''
                  python3 -m venv $VENV_DIR
                  . $VENV_DIR/bin/activate
                  pip install --upgrade pip
                  pip install -r requirements.txt
                  ansible-galaxy collection install amazon.aws
                '''
            }
        }
        stage('Run Ansible Playbook') {
            steps {
                sh '''
                  . $VENV_DIR/bin/activate
                  ansible-playbook -i inventory/hosts playbooks/timeout-s3-bucket.yml
                '''
            }
        }
    }
    post {
        success {
            echo 'S3 bucket created successfully via Jenkins/Ansible!'
        }
        failure {
            echo 'S3 bucket deployment failed.'
        }
    }
}
