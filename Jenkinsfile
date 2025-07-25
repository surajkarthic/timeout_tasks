pipeline {
    agent any

    // Add a parameter to let you choose the action
    parameters {
        choice(name: 'ACTION', choices: ['create', 'delete'], description: 'Choose the action to perform.')
    }

    environment {
        VENV_DIR = 'venv'
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

        // This stage ONLY runs if you select 'create'
        stage('Create S3 Bucket') {
            when {
                expression { params.ACTION == 'create' }
            }
            steps {
                sh '''
                    . $VENV_DIR/bin/activate
                    ansible-playbook -i inventory/hosts playbooks/timeout-s3-bucket.yml
                '''
            }
        }

        // This stage ONLY runs if you select 'delete'
        stage('Delete All S3 Buckets') {
            when {
                expression { params.ACTION == 'delete' }
            }
            steps {
                sh '''
                    . $VENV_DIR/bin/activate
                    # Run the new multi-region playbook and pass the regions
                    ansible-playbook playbooks/delete-buckets-multi-region.yml --extra-vars "aws_regions=['ap-south-1','us-east-1']"
                '''
            }
        }
    }

    post {
        success {
            echo "Action '${params.ACTION}' completed successfully!"
        }
        failure {
            echo "Action '${params.ACTION}' failed."
        }
    }
}
