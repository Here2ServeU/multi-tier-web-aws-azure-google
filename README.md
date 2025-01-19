# Automated Multi-Cloud Deployment Project

## Project Overview

This project aims to create a fully automated pipeline to deploy a multi-tier web application on AWS, Azure, and Google Cloud using Terraform and CI/CD pipelines. The infrastructure includes:
- AWS: EC2 instances and S3 buckets
- Azure: Virtual Machines (VMs) and Blob Storage
- Google Cloud: Compute Engine instances and Cloud Storage

## Features:
- Reusable Terraform modules for managing resources across cloud providers.
- Secure secrets management using AWS Secrets Manager, Azure Key Vault, and Google Cloud Secret Manager.
- Deployment pipeline using Jenkins or GitHub Actions.

---

## Project Structure
```txt
multi-cloud-deployment/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── outputs.tf
│   ├── stage/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── outputs.tf
│   ├── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       ├── outputs.tf
├── modules/
│   ├── aws/
│   │   ├── ec2/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── outputs.tf
│   │   ├── s3/
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       ├── outputs.tf
│   ├── azure/
│       ├── vm/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   ├── outputs.tf
│       ├── blob/
│           ├── main.tf
│           ├── variables.tf
│           ├── outputs.tf
│   ├── gcp/
│       ├── compute/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   ├── outputs.tf
│       ├── storage/
│           ├── main.tf
│           ├── variables.tf
│           ├── outputs.tf
├── pipelines/
│   ├── Jenkinsfile
│   ├── github-actions.yml
├── README.md
```

## Terraform Modules

### AWS EC2 Module

#### File: modules/aws/ec2/main.tf
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  tags = {
    Name = var.name
  }
}

output "instance_id" {
  value = aws_instance.web.id
}
```

#### File: modules/aws/ec2/variables.tf
```hcl
variable "ami_id" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "name" {
  type = string
}
```

#### File: modules/aws/ec2/outputs.tf
```hcl
output "ec2_instance_id" {
  value = aws_instance.web.id
}
```

### Azure VM Module

#### File: modules/azure/vm/main.tf
```hcl
resource "azurerm_virtual_machine" "vm" {
  name                  = var.name
  location              = var.location
  resource_group_name   = var.resource_group_name
  network_interface_ids = [var.network_interface_id]
  vm_size               = var.vm_size
  delete_os_disk_on_termination = true

  os_profile {
    computer_name  = var.name
    admin_username = var.admin_username
    admin_password = var.admin_password
  }
}
```

#### File: modules/azure/vm/variables.tf
```hcl
variable "name" {
  type = string
}

variable "location" {
  type = string
}

variable "resource_group_name" {
  type = string
}

variable "network_interface_id" {
  type = string
}

variable "vm_size" {
  type = string
}

variable "admin_username" {
  type = string
}

variable "admin_password" {
  type = string
}
```

#### File: modules/azure/vm/outputs.tf
```hcl 
output "vm_id" {
  value = azurerm_virtual_machine.vm.id
}
```

### Google Cloud Compute Module

#### File: modules/gcp/compute/main.tf
```hcl
resource "google_compute_instance" "default" {
  name         = var.name
  machine_type = var.machine_type
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = var.image
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }
}
```

#### File: modules/gcp/compute/variables.tf
```hcl
variable "name" {
  type = string
}

variable "machine_type" {
  type = string
}

variable "zone" {
  type = string
}

variable "image" {
  type = string
}
```

#### File: modules/gcp/compute/outputs.tf
```hcl
output "instance_name" {
  value = google_compute_instance.default.name
}
```

---

## CI/CD Pipeline

### Jenkinsfile
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -var-file=terraform.tfvars'
            }
        }
        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve -var-file=terraform.tfvars'
            }
        }
    }
}
```

### GitHub Actions Workflow

#### File: .github/workflows/deploy.yml
```yml
name: Deploy Multi-Cloud

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Terraform
      uses: hashicorp/setup-terraform@v2

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: terraform plan -var-file=terraform.tfvars

    - name: Terraform Apply
      run: terraform apply -auto-approve -var-file=terraform.tfvars
```

---

## Steps to Use AWS Secrets Manager, Azure Key Vault, and Google Cloud Secrets Manager

### 1. AWS Secrets Manager

#### Step 1: Create a Secret
1.	Go to the AWS Secrets Manager Console.

2.	Click Store a new secret.

3.	Choose the secret type:
- Credentials (e.g., database username and password).

4.	Enter the key-value pairs for the secret.

5.	Select the encryption key (default AWS KMS or a custom KMS key).

6.	Name the secret (e.g., db-credentials).

7.	Finish and save the secret.

#### Step 2: Access Secrets in Terraform

- Add the following to the Terraform configuration:
```hcl
provider "aws" {
  region = "us-east-1"
}

data "aws_secretsmanager_secret" "db_secret" {
  name = "db-credentials"
}

data "aws_secretsmanager_secret_version" "db_secret_version" {
  secret_id = data.aws_secretsmanager_secret.db_secret.id
}

output "db_credentials" {
  value = jsondecode(data.aws_secretsmanager_secret_version.db_secret_version.secret_string)
}
```
- Use jsondecode() to extract specific key-value pairs, such as username and password.

#### Step 3: Secure Access

1.	Assign appropriate IAM policies to the Terraform execution role.

2.	Example policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-credentials"
    }
  ]
}
```

### 2. Azure Key Vault

#### Step 1: Create a Key Vault
1.	Go to the Azure Portal.

2.	Navigate to Key Vaults → + Create.

3.	Configure:
- Resource Group: Select or create one.
- Vault Name: Example, key-vault-dev.
- Region: Select a region.
- Enable soft delete for recovery.

#### Step 2: Add a Secret
1.	Open the Key Vault.

2.	Navigate to Secrets → + Generate/Import.

3.	Enter:
- Name: Example, db-credentials.
- Value: Example, {"username":"admin", "password":"securepassword"}.

4.	Save the secret.

#### Step 3: Access Secrets in Terraform

- Add the following to the Terraform configuration:
```hcl
provider "azurerm" {
  features {}
}

data "azurerm_key_vault" "example" {
  name                = "key-vault-dev"
  resource_group_name = "your-resource-group"
}

data "azurerm_key_vault_secret" "db_secret" {
  name         = "db-credentials"
  key_vault_id = data.azurerm_key_vault.example.id
}

output "db_credentials" {
  value = jsondecode(data.azurerm_key_vault_secret.db_secret.value)
}
```

#### Step 4: Secure Access
1.	Assign the Azure Key Vault Reader Role to the Terraform service principal.

2.	Use the following CLI command:
```bash
az role assignment create --role "Key Vault Reader" --assignee <service-principal-id> --scope <key-vault-id>
```

### 3. Google Cloud Secrets Manager

#### Step 1: Create a Secret
1.	Go to the Google Cloud Console.

2.	Navigate to Secrets Manager → + Create Secret.

3.	Configure:
- Name: Example, db-credentials.
- Value: Example, {"username":"admin", "password":"securepassword"}.

4.	Save the secret.

#### Step 2: Access Secrets in Terraform

- Add the following to the Terraform configuration:
```hcl
provider "google" {
  project = "your-gcp-project-id"
}

data "google_secret_manager_secret" "db_secret" {
  secret_id = "db-credentials"
}

data "google_secret_manager_secret_version" "db_secret_version" {
  secret    = data.google_secret_manager_secret.db_secret.id
  version   = "latest"
}

output "db_credentials" {
  value = jsondecode(data.google_secret_manager_secret_version.db_secret_version.secret_data)
}
```

#### Step 3: Secure Access
1.	Assign the Secret Manager Accessor Role to the service account running Terraform.

2.	Example command:
```bash
gcloud projects add-iam-policy-binding your-gcp-project-id \
  --member="serviceAccount:your-service-account@your-project.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### Pipeline Integration
#### 1.	Store Terraform Variables Securely:
- Pass the secrets as environment variables to the CI/CD pipeline.
- Example in Jenkinsfile:
```groovy
environment {
    DB_CREDENTIALS = credentials('aws-secrets-manager-id') // Use Jenkins credentials plugin
}
```

- Example in GitHub Actions:
```yml
env:
  DB_CREDENTIALS: ${{ secrets.AWS_DB_CREDENTIALS }}
```

#### 2.	Inject Secrets into Terraform:
- Use Terraform variable files (terraform.tfvars) with placeholders for secret values.
- Inject secrets at runtime via environment variables.

### Key Considerations
- IAM Roles: Ensure Terraform execution roles have the least privilege to access secrets.
- Audit Logging: Enable logging for Secrets Manager, Key Vault, and GCP Secrets Manager to monitor access.
- Versioning: Use secret versioning for rotation and rollback.

--- 

## Clean up

