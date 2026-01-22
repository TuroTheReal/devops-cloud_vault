# Terraform - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, terraform, iac]
created: 2025-01-21
version: 1.x
```

**Official docs**: [terraform.io/docs](https://developer.hashicorp.com/terraform/docs)
**Related concepts**: [[concepts/terraform/terraform-fundamentals]]

---

## 📑 Table of Contents

1. [Quick Start](#-quick-start)
2. [Core Workflow](#-core-workflow)
3. [State Management](#-state-management)
4. [Debug / Inspection](#-debug--inspection)
5. [Useful One-Liners](#-useful-one-liners)
6. [Config Example](#-config-example)

---

## 🚀 Quick Start

```bash
# Installation (Ubuntu)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Basic workflow
terraform init      # Prepare project
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Delete everything
```

---

## 🔄 Core Workflow

### Init
```bash
# Initialize project (download providers)
terraform init

# Reinitialize and upgrade providers
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure
```

### Plan
```bash
# Preview planned changes
terraform plan

# Save plan to file
terraform plan -out=plan.tfplan

# Plan with variable
terraform plan -var="instance_type=t3.small"
```

### Apply
```bash
# Apply changes (with confirmation)
terraform apply

# Apply without confirmation
terraform apply -auto-approve

# Apply saved plan
terraform apply plan.tfplan

# Apply with variable
terraform apply -var="instance_type=t3.small"
```

### Destroy
```bash
# Destroy all infrastructure
terraform destroy

# Destroy without confirmation
terraform destroy -auto-approve

# Destroy specific resource
terraform destroy -target=aws_instance.web
```

---

## 📦 State Management

```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Remove resource from state (without destroying on AWS)
terraform state rm aws_instance.web

# Move/rename resource
terraform state mv aws_instance.old aws_instance.new

# Import existing resource into state
terraform import aws_instance.web i-1234567890abcdef0

# Force unlock state (if locked)
terraform force-unlock LOCK_ID
```

---

## 🔍 Debug / Inspection

### Validation
```bash
# Validate syntax
terraform validate

# Format code
terraform fmt

# Format recursively
terraform fmt -recursive

# Check formatting (CI)
terraform fmt -check
```

### Outputs
```bash
# Show all outputs
terraform output

# Show specific output
terraform output public_ip

# Output as JSON
terraform output -json
```

### Debug
```bash
# Verbose debug mode
TF_LOG=DEBUG terraform apply

# Logs to file
TF_LOG=DEBUG TF_LOG_PATH=./terraform.log terraform apply
```

---

## ⚡ Useful One-Liners

```bash
# Preview what will be destroyed
terraform plan -destroy

# Refresh state without applying
terraform refresh

# List installed providers
terraform providers

# Dependency graph (generates DOT)
terraform graph | dot -Tpng > graph.png

# Target specific resource
terraform apply -target=aws_instance.web

# Replace resource (force recreate)
terraform apply -replace=aws_instance.web
```

---

## 📝 Config Example

### main.tf
```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # S3 backend for state (prod/team)
  # backend "s3" {
  #   bucket = "my-tfstate-bucket"
  #   key    = "prod/terraform.tfstate"
  #   region = "eu-west-1"
  # }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name    = "web-server"
    Project = var.project_name
  }
}
```

### variables.tf
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "ami_id" {
  description = "AMI ID"
  type        = string
}

variable "project_name" {
  description = "Project name for tagging"
  type        = string
  default     = "my-project"
}
```

### outputs.tf
```hcl
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP address"
  value       = aws_instance.web.public_ip
}
```

### terraform.tfvars
```hcl
aws_region    = "eu-west-1"
instance_type = "t3.micro"
ami_id        = "ami-0c55b159cbfafe1f0"
project_name  = "my-project"
```

---

**Last update**: 2025-01-21
**Tool version**: 1.x
