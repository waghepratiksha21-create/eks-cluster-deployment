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

        stage('Import Existing Resources') {
            steps {
                script {

                    // IAM Roles
                    ['eks-cluster-example-2':'example', 'eks-node-role-2':'worker'].each { roleName, tfRes ->

                        def inState = sh(script: "terraform state list | grep aws_iam_role.${tfRes} || true", returnStdout:true).trim()

                        if(!inState) {
                            def exists = sh(script: "aws iam get-role --role-name ${roleName} >/dev/null 2>&1 && echo true || echo false", returnStdout:true).trim()

                            if(exists == 'true') {
                                echo "Importing IAM role ${roleName}..."
                                sh "terraform import aws_iam_role.${tfRes} ${roleName}"
                            }
                        } else {
                            echo "IAM role ${roleName} already managed by Terraform"
                        }
                    }

                    // EKS Cluster
                    def clusterState = sh(script: "terraform state list | grep aws_eks_cluster.project-cluster || true", returnStdout:true).trim()

                    if(!clusterState){
                        def clusterExists = sh(script:"aws eks describe-cluster --name project-cluster >/dev/null 2>&1 && echo true || echo false", returnStdout:true).trim()

                        if(clusterExists == 'true'){
                            echo "Importing existing EKS cluster..."
                            sh "terraform import aws_eks_cluster.project-cluster project-cluster"
                        }
                    } else {
                        echo "EKS cluster already managed by Terraform"
                    }

                    // Node Group
                    def nodeState = sh(script: "terraform state list | grep aws_eks_node_group.node-grp || true", returnStdout:true).trim()

                    if(!nodeState){
                        def nodeGroupExists = sh(script:"aws eks describe-nodegroup --cluster-name project-cluster --nodegroup-name pc-node-group >/dev/null 2>&1 && echo true || echo false", returnStdout:true).trim()

                        if(nodeGroupExists == 'true'){
                            echo "Importing existing EKS node group..."
                            sh "terraform import aws_eks_node_group.node-grp project-cluster/pc-node-group"
                        }
                    } else {
                        echo "Node group already managed by Terraform"
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
