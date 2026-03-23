pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['apply', 'destroy'],
            description: 'Select the action to perform'
        )
    }

    environment {
        TF_VAR_aws_region = 'ap-south-1'  // Optional: set your AWS region
    }

    stages {

        stage('Terraform Init') {
            steps {
                sh 'terraform init -reconfigure'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan'
            }
        }

        stage('Handle Existing IAM Roles') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                script {
                    echo "Checking for existing IAM roles..."
                    // This avoids failure if role exists
                    def roles = ['eks-cluster-example-2', 'eks-node-role-2']
                    roles.each { role ->
                        def exists = sh(script: "aws iam get-role --role-name ${role} > /dev/null 2>&1 && echo true || echo false", returnStdout: true).trim()
                        if (exists == 'true') {
                            echo "Role ${role} already exists. Importing to Terraform..."
                            sh "terraform import aws_iam_role.${role.replace('-', '_')} ${role} || echo 'Already imported'"
                        }
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    if (params.ACTION == 'apply') {
                        echo 'Executing Terraform Apply...'
                        sh 'terraform apply --auto-approve'
                    } else if (params.ACTION == 'destroy') {
                        echo 'Executing Terraform Destroy...'
                        sh 'terraform destroy --auto-approve'
                    }
                }
            }
        }

    }
}
