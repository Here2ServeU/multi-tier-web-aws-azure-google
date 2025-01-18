
# Automated Multi-Cloud Deployment Project

## Project Overview

This project aims to create a fully automated pipeline to deploy a multi-tier web application on AWS, Azure, and Google Cloud using Terraform and CI/CD pipelines. The infrastructure includes:
- AWS: EC2 instances and S3 buckets
- Azure: Virtual Machines (VMs) and Blob Storage
- Google Cloud: Compute Engine instances and Cloud Storage

## Features:
- Reusable Terraform modules for managing resources across cloud providers.
- Secure secrets management using AWS Secrets Manager and Azure Key Vault.
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

## Clean up

