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
        TF_VAR_aws_region = 'ap-south-1'
    }

    stages {

        stage('Terraform Init') {
            steps {
                echo 'Initializing Terraform...'
                sh 'terraform init -reconfigure'
            }
        }

        stage('Terraform Plan') {
            steps {
                echo 'Running Terraform Plan...'
                sh 'terraform plan'
            }
        }

        stage('Handle Existing IAM Roles') {
            when { expression { params.ACTION == 'apply' } }
            steps {
                script {
                    echo "Checking for existing IAM roles..."
                    def roleMap = [
                        'eks-cluster-example-2': 'example',
                        'eks-node-role-2'      : 'worker'
                    ]
                    roleMap.each { awsRoleName, tfResourceName ->
                        def exists = sh(script: "aws iam get-role --role-name ${awsRoleName} > /dev/null 2>&1 && echo true || echo false", returnStdout: true).trim()
                        if (exists == 'true') {
                            echo "Importing role ${awsRoleName}..."
                            sh "terraform import aws_iam_role.${tfResourceName} ${awsRoleName} || echo 'Already imported'"
                        }
                    }
                }
            }
        }

        stage('Handle Existing EKS Cluster & Node Group') {
            when { expression { params.ACTION == 'apply' } }
            steps {
                script {
                    echo "Checking for existing EKS cluster..."
                    def clusterExists = sh(script: "aws eks describe-cluster --name project-cluster > /dev/null 2>&1 && echo true || echo false", returnStdout: true).trim()
                    if (clusterExists == 'true') {
                        echo "Importing existing EKS cluster..."
                        sh "terraform import aws_eks_cluster.project-cluster project-cluster || echo 'Already imported'"
                    }

                    echo "Checking for existing EKS node group..."
                    def nodeGroupExists = sh(script: "aws eks describe-nodegroup --cluster-name project-cluster --nodegroup-name pc-node-group > /dev/null 2>&1 && echo true || echo false", returnStdout: true).trim()
                    if (nodeGroupExists == 'true') {
                        echo "Importing existing EKS node group..."
                        sh "terraform import aws_eks_node_group.node-grp project-cluster/pc-node-group || echo 'Already imported'"
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
        success { echo 'Pipeline finished successfully.' }
        failure { echo 'Terraform action failed. Check logs for details.' }
    }
}
