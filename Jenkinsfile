pipeline {
     agent {
        label 'public-ec2-slave'
    }

    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Select The Action To Perform')
        choice(name: 'WORKSPACE', choices: ['dev', 'prod'], description: 'Terraform Workspace Choice')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID ')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Clone repo') {
            steps {
                git url: 'https://github.com/AlaaGomaa178/CI-CD-Pipeline-with-Jenkins-Terraform-and-Ansible-.git', branch: 'main'
                
            }
        }

        stage('Terraform init'){
            steps {
                dir('TerraformCode') {
                    sh 'terraform init'
                }
            }
        }

        stage('Create or Select Terraform Workspace') {
            steps {
                script {
                    try {
                        dir('TerraformCode') {
                            sh "terraform workspace select ${params.WORKSPACE}"
                        }
                    } catch (Exception e) {
                        dir('TerraformCode') {
                            sh "terraform workspace new ${params.WORKSPACE}"
                            sh "terraform workspace show"
                        }
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('TerraformCode') {
                    sh "terraform plan -var-file=${params.WORKSPACE}-variables.tfvars"
                }
            }
        }

        stage('Terraform Apply/Destroy') {
            steps {
                script {
                    dir('TerraformCode') {
                        if (params.ACTION == 'apply') {
                            sh "terraform ${params.ACTION} -auto-approve -var-file=${params.WORKSPACE}-variables.tfvars"
                        } else if (params.ACTION == 'destroy') {
                            sh "terraform ${params.ACTION} -auto-approve -var-file=${params.WORKSPACE}-variables.tfvars"
                        } else {
                            error "Invalid action selected. Please choose either 'apply' or 'destroy'."
                        }
                    }
                }
            }
        }
    }
}
