# CI/CD Pipeline with Jenkins, Terraform, and Ansible

## Overview
This repository provides a comprehensive CI/CD pipeline that integrates Jenkins, Terraform, and Ansible. The pipeline automates the provisioning of infrastructure on AWS, deployment of a Node.js application, and configuration management of Jenkins master and slave nodes.

## Repository Structure
The repository is organized as follows:

```
├── Ansible
│   ├── configure-jenkins-master
│   │   └── playbook.yaml
│   ├── configure-jenkins-private-slave
│   │   └── playbook.yaml
│   ├── configure-jenkins-public-slave
│   │   └── playbook.yaml
│   ├── inventory
│   └── playbook.yaml
├── Jenkinsfile
├── NodejsApp_Jenkinsfile
├── Terraform
│   ├── backend.tf
│   ├── dev.tfvars
│   ├── instances.tf
│   ├── keys.tf
│   ├── lambda-python-code.py
│   ├── lambda.tf
│   ├── network
│   │   ├── igw.tf
│   │   ├── natgw.tf
│   │   ├── outputs.tf
│   │   ├── rt-route-rtassociate.tf
│   │   ├── sn.tf
│   │   ├── variables.tf
│   │   ├── vpc.tf
│   ├── network.tf
│   ├── prod.tfvars
│   ├── provider.tf
│   ├── rds.tf
│   ├── redis_elastic_cache.tf
│   ├── role.tf
│   ├── security_groups.tf
│   ├── ses.tf
│   ├── terraform.tfvars
│   ├── variables.tf
└── README.md
```

## Jenkins Pipelines

### Main Jenkinsfile
This Jenkinsfile automates the Terraform infrastructure deployment and management.

#### Parameters
- **ACTION**: Select the action to perform (`apply` or `destroy`).
- **WORKSPACE**: Choose the Terraform workspace (`dev` or `prod`).

#### Environment Variables
- **AWS_ACCESS_KEY_ID**: AWS access key ID.
- **AWS_SECRET_ACCESS_KEY**: AWS secret access key.
- **AWS_DEFAULT_REGION**: Default AWS region (`us-east-1`).

#### Stages
1. **Clone repo**: Clones the repository from GitHub.
2. **Terraform init**: Initializes the Terraform configuration.
3. **Create or Select Terraform Workspace**: Selects or creates the Terraform workspace.
4. **Terraform Plan**: Runs the Terraform plan.
5. **Terraform Apply/Destroy**: Applies or destroys the Terraform configuration based on the selected action.

### NodejsApp_Jenkinsfile
This Jenkinsfile automates the Docker build and deployment of a Node.js application.

#### Environment Variables
- **RDS_HOSTNAME**: RDS hostname.
- **RDS_USERNAME**: RDS username.
- **RDS_PASSWORD**: RDS password.
- **RDS_PORT**: RDS port.
- **REDIS_HOSTNAME**: Redis hostname.
- **REDIS_PORT**: Redis port.

#### Stages
1. **Clone Github Repo**: Clones the Node.js application repository.
2. **Docker Build**: Builds the Docker image.
3. **Docker Run => Deploy the app**: Deploys the Docker container.

## Ansible Playbooks

### configure-jenkins-master
Configures the Jenkins master node.

#### Tasks
- Update the apt cache.
- Install OpenJDK 11.
- Add Jenkins repository key and APT repository.
- Install Jenkins.
- Ensure Jenkins is running.

### configure-jenkins-private-slave
Configures the Jenkins private slave node.

#### Tasks
- Update the apt cache.
- Install Java.
- Create a directory for the Jenkins slave.
- Install Docker.
- Add the ubuntu user to the Docker group.

### configure-jenkins-public-slave
Configures the Jenkins public slave node.

#### Tasks
- Update the apt cache.
- Install Java.
- Create a directory for the Jenkins slave.
- Install required packages for Terraform.
- Add HashiCorp GPG key and repository.
- Install Terraform.

### Inventory
Defines the inventory of the Jenkins master and slave nodes.

## Terraform Files

### backend.tf
Configures the S3 backend for storing the Terraform state and DynamoDB for state locking.

### dev.tfvars & prod.tfvars
Contains environment-specific variables for the `dev` and `prod` environments.

### instances.tf
Defines the AWS EC2 instances for the bastion and application servers.

### keys.tf
Generates and manages the SSH keys.

### lambda.tf & lambda-python-code.py
Defines the AWS Lambda function for sending SES emails and the Python code for the Lambda function.

### network
Contains the Terraform code for creating the VPC, subnets, internet gateway, NAT gateway, route tables, and associations.

### network.tf
Defines the network module that includes all the network-related resources.

### provider.tf
Configures the AWS provider.

### rds.tf
Defines the RDS instance and its subnet group.

### redis_elastic_cache.tf
Defines the Redis Elasticache cluster and its subnet group.

### role.tf
Defines the IAM role and policy for the Lambda function.

### security_groups.tf
Defines the security groups for the public and private instances.

### ses.tf
Defines the SES email identity.

### variables.tf
Contains the variable definitions used throughout the Terraform configuration.

## Setting Up the Environment

### Prerequisites
- **Jenkins server**: Jenkins server with necessary plugins installed (Pipeline, Git, Ansible, etc.).
- **AWS Account**: AWS account with appropriate IAM permissions.
- **SSH Key Pair**: SSH key pair for accessing EC2 instances.
- **Docker**: Docker installed on the Jenkins slave nodes.

### Steps

1. **Configure Jenkins Master and Slaves**: Use Ansible playbooks to set up Jenkins master and slave nodes.
   - Run the Ansible playbook for the master node:
     ```sh
     ansible-playbook -i inventory configure-jenkins-master/playbook.yaml
     ```
   - Run the Ansible playbook for the public slave node:
     ```sh
     ansible-playbook -i inventory configure-jenkins-public-slave/playbook.yaml
     ```
   - Run the Ansible playbook for the private slave node:
     ```sh
     ansible-playbook -i inventory configure-jenkins-private-slave/playbook.yaml
     ```

2. **Set Up Jenkins Jobs**:
   - Create a Jenkins pipeline job using the `Jenkinsfile` to manage the infrastructure.
     - In Jenkins, go to `New Item`.
     - Select `Pipeline` and give it a name.
     - In the pipeline configuration, select `Pipeline script from SCM`.
     - Choose `Git` and provide the repository URL.
     - Specify the script path as `Jenkinsfile`.
   - Create another Jenkins pipeline job using the `NodejsApp_Jenkinsfile` to deploy the Node.js application.
     - Repeat the steps above, specifying the script path as `NodejsApp_Jenkinsfile`.

3. **Configure Terraform Backend**:
   - Ensure the S3 bucket and DynamoDB table specified in `backend.tf` exist.
   - Initialize the Terraform backend:
     ```sh
     terraform init
     ```

4. **Deploy the Infrastructure**:
   - Trigger the Jenkins job with the appropriate parameters (`apply` or `destroy`, `dev` or `prod`) to manage the infrastructure.
     - For example, to apply the `dev` environment:
       ```sh
       ACTION=apply WORKSPACE=dev
       ```

5. **Deploy the Node.js Application**:
   - Trigger the Jenkins job to build and deploy the Node.js application using Docker.
     - Make sure the environment variables (RDS and Redis details) are correctly set.

## Troubleshooting

### Common Issues

1. **Terraform Backend Initialization**:
   - Ensure that the S3 bucket and DynamoDB table specified in `backend.tf` are created and accessible.
   - Verify the AWS credentials and region settings.

2. **Jenkins Slave Configuration**:
   - Ensure that the Jenkins master can connect to the slave nodes.
   - Check the Docker installation and permissions on the slave nodes.

3. **Application Deployment**:
   - Ensure that the Node.js application repository URL is correct.
   - Verify the Dockerfile and environment variable configurations.

## Conclusion
This repository provides a full-fledged CI/CD pipeline with Jenkins, Terraform, and Ansible, enabling automated infrastructure provisioning, configuration management, and application deployment. Customize the configuration files and scripts as per your requirements to extend and adapt this solution to your specific use case.
