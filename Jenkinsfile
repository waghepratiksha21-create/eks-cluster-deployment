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
        TF_VAR_aws_region = 'ap-south-1'  // Set your AWS region
    }

    stages {

        stage('Terraform Init') {
            steps {
                echo "Initializing Terraform..."
                sh 'terraform init -reconfigure'
            }
        }

        stage('Terraform Plan') {
            steps {
                echo "Running Terraform Plan..."
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

                    // Map AWS role names to Terraform resource names
                    def roleMap = [
                        'eks-cluster-example-2': 'eks_cluster_example_2',
                        'eks-node-role-2'      : 'eks_node_role_2'
                    ]

                    roleMap.each { awsRoleName, tfResourceName ->
                        def exists = sh(
                            script: "aws iam get-role --role-name ${awsRoleName} > /dev/null 2>&1 && echo true || echo false",
                            returnStdout: true
                        ).trim()

                        if (exists == 'true') {
                            echo "Role ${awsRoleName} already exists. Importing to Terraform..."
                            // Import safely; ignore errors if already imported
                            sh "terraform import aws_iam_role.${tfResourceName} ${awsRoleName} || echo 'Already imported'"
                        } else {
                            echo "Role ${awsRoleName} does not exist. Terraform will create it."
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

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Terraform ${params.ACTION} completed successfully."
        }
        failure {
            echo "Terraform ${params.ACTION} failed. Check logs for details."
        }
    }
}
