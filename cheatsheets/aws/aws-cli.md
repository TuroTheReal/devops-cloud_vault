# AWS CLI - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, aws, cli]
created: 2025-01-21
version: 2.x
```

**Official docs**: [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/)
**Related concepts**: [[concepts/aws/aws-fundamentals]]

---

## 📑 Table of Contents

1. [Quick Start](#-quick-start)
2. [Configuration](#-configuration)
3. [EC2](#-ec2)
4. [S3](#-s3)
5. [IAM](#-iam)
6. [Logs (CloudWatch)](#-logs-cloudwatch)
7. [Useful One-Liners](#-useful-one-liners)

---

## 🚀 Quick Start

```bash
# Installation (Ubuntu)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Initial configuration
aws configure

# Verify identity
aws sts get-caller-identity
```

---

## ⚙️ Configuration

```bash
# Interactive configuration
aws configure

# Configure specific profile
aws configure --profile prod

# Use a profile
aws ec2 describe-instances --profile prod
# or
export AWS_PROFILE=prod

# List configurations
aws configure list

# Get current region
aws configure get region
```

### ~/.aws/credentials
```ini
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

[prod]
aws_access_key_id = AKIA...
aws_secret_access_key = ...
```

### ~/.aws/config
```ini
[default]
region = us-east-1
output = json

[profile prod]
region = eu-west-1
output = table
```

---

## 🖥️ EC2

### Instances
```bash
# List all instances
aws ec2 describe-instances --output table

# List with filters (by tag)
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=web*" \
  --output table

# List running instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running"

# Compact format (ID + State + Name)
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key=='Name'].Value|[0]]" \
  --output table

# Start an instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop an instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Terminate (delete) an instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Reboot
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

### Security Groups
```bash
# List Security Groups
aws ec2 describe-security-groups --output table

# Show SG details
aws ec2 describe-security-groups --group-ids sg-12345678

# Create a SG
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group"

# Add ingress rule (SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr X.X.X.X/32
```

### Key Pairs
```bash
# List key pairs
aws ec2 describe-key-pairs

# Create a key pair
aws ec2 create-key-pair \
  --key-name my-key \
  --query 'KeyMaterial' \
  --output text > my-key.pem

# Delete
aws ec2 delete-key-pair --key-name my-key
```

### AMIs
```bash
# Search Ubuntu AMI (latest)
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd*/ubuntu-*-24.04-amd64-server-*" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].[ImageId,Name]' \
  --output table
```

---

## 📦 S3

```bash
# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://my-bucket/

# List recursively
aws s3 ls s3://my-bucket/ --recursive

# Upload a file
aws s3 cp file.txt s3://my-bucket/

# Upload a folder
aws s3 cp ./folder s3://my-bucket/folder --recursive

# Sync (like rsync)
aws s3 sync ./local-folder s3://my-bucket/remote-folder

# Download
aws s3 cp s3://my-bucket/file.txt ./

# Delete
aws s3 rm s3://my-bucket/file.txt

# Delete recursively
aws s3 rm s3://my-bucket/folder/ --recursive

# Create a bucket
aws s3 mb s3://my-new-bucket

# Delete a bucket (must be empty)
aws s3 rb s3://my-bucket

# Delete bucket + contents
aws s3 rb s3://my-bucket --force
```

---

## 🔐 IAM

```bash
# Who am I?
aws sts get-caller-identity

# List users
aws iam list-users

# List roles
aws iam list-roles

# List policies attached to a user
aws iam list-attached-user-policies --user-name myuser

# List user's groups
aws iam list-groups-for-user --user-name myuser
```

---

## 📊 Logs (CloudWatch)

```bash
# List log groups
aws logs describe-log-groups

# List log streams of a group
aws logs describe-log-streams \
  --log-group-name /aws/lambda/my-function

# View latest logs
aws logs tail /aws/lambda/my-function

# Follow logs (live)
aws logs tail /aws/lambda/my-function --follow

# Filter logs
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR"
```

---

## ⚡ Useful One-Liners

```bash
# View all instances with their public IPs
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress,Tags[?Key=='Name'].Value|[0]]" \
  --output table

# Find instances by region
for region in $(aws ec2 describe-regions --query "Regions[*].RegionName" --output text); do
  echo "=== $region ==="
  aws ec2 describe-instances --region $region \
    --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
    --output table
done

# Cost of running instances (approximate)
aws ce get-cost-and-usage \
  --time-period Start=2025-01-01,End=2025-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost

# Export SG rules to JSON
aws ec2 describe-security-groups \
  --query "SecurityGroups[*].{Name:GroupName,Rules:IpPermissions}" \
  --output json > sg-rules.json
```

---

**Last update**: 2025-01-21
**Tool version**: 2.x
