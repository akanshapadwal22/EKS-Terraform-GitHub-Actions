properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'],
            name: 'Terraform_Action'
        )
    ])
])

pipeline {
    agent any
    environment {
        TERRAFORM_VERSION = '1.9.3'
        TERRAFORM_DIR = "${env.WORKSPACE}/terraform-bin"
    }
    stages {

        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }

        stage('Git Pulling') {
            steps {
                git branch: 'master', url: 'https://github.com/akanshapadwal22/EKS-Terraform-GitHub-Actions.git'
            }
        }

        stage('Install Terraform') {
            steps {
                sh '''
                    echo "Installing Terraform ${TERRAFORM_VERSION} locally..."
                    mkdir -p ${TERRAFORM_DIR}
                    curl -Lo ${TERRAFORM_DIR}/terraform.zip https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip
                    unzip -o ${TERRAFORM_DIR}/terraform.zip -d ${TERRAFORM_DIR}
                    chmod +x ${TERRAFORM_DIR}/terraform
                    export PATH=${TERRAFORM_DIR}:$PATH
                    terraform version
                '''
            }
        }

        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh '''
                        export PATH=${TERRAFORM_DIR}:$PATH
                        terraform -chdir=eks/ init
                    '''
                }
            }
        }

        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh '''
                        export PATH=${TERRAFORM_DIR}:$PATH
                        terraform -chdir=eks/ validate
                    '''
                }
            }
        }

        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {
                        def tfCmd = "terraform -chdir=eks/ ${params.Terraform_Action} -var-file=${params.Environment}.tfvars"
                        if (params.Terraform_Action == 'apply' || params.Terraform_Action == 'destroy') {
                            tfCmd += " -auto-approve"
                        }
                        sh """
                            export PATH=${TERRAFORM_DIR}:$PATH
                            ${tfCmd}
                        """
                    }
                }
            }
        }
    }
}
