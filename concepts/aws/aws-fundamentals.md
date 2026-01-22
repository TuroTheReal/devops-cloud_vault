# AWS Fundamentals - Cloud Services Overview

## 📋 Metadata

```yaml
tags: [concept, aws, cloud, status/learning]
created: 2025-01-21
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 20h
```

**Prerequisites**: [[concepts/networking/networking-fundamentals]]
**Related to**: [[concepts/terraform/terraform-fundamentals]]
**Official docs**: [AWS Docs](https://docs.aws.amazon.com/) | [AWS CLI Reference](https://docs.aws.amazon.com/cli/)

---

## 🎯 TL;DR (30 seconds)

AWS is a cloud provider offering on-demand services (servers, storage, networking, databases...). You pay for what you use, you scale based on your needs.

**Analogy**: AWS = an infinite datacenter where you rent exactly what you need, when you need it.

---

## 🤔 When to Use?

### ✅ Good for
1. **Scalability**: From 1 to 1000 servers based on load
2. **No hardware**: No upfront investment, no physical maintenance
3. **Managed services**: Databases, cache, queues... without managing servers

### ❌ Bad for
- **Very tight budget** → Classic VPS cheaper for small projects
- **Highly sensitive data** → On-premise may be required (compliance)

---

## 📚 Key Concepts

### 1. Main Services (overview)

**My understanding**:

| Service | Category | Simple Description |
|---------|----------|-------------------|
| **EC2** | Compute | Virtual servers (AWS VPS) |
| **S3** | Storage | Unlimited file storage |
| **VPC** | Network | Virtual private network |
| **RDS** | Database | Managed databases (MySQL, PostgreSQL...) |
| **ECS** | Containers | Simple Docker orchestration |
| **EKS** | Containers | Managed Kubernetes |
| **Lambda** | Serverless | Code without servers (pay-per-execution) |
| **CloudWatch** | Monitoring | Logs and metrics |
| **Route 53** | DNS | DNS + routing |
| **ALB/ELB** | Load Balancing | Distribute traffic across instances (L7/L4) |

**Why important**:
Knowing the services prevents reinventing the wheel. Need a database? RDS. Need file storage? S3.

**Docs**: [All AWS Services](https://aws.amazon.com/products/)

---

### 2. IAM (Identity and Access Management)

**My understanding**:
IAM manages WHO can do WHAT on AWS. Three key concepts:
- **Users**: Physical people (you, your colleagues) - like Linux users
- **Groups**: Collection of users sharing permissions - like Linux groups
- **Roles**: **Temporary** identities for services (EC2 accessing S3) or cross-account access - like `sudo -u` (assume another identity temporarily)
- **Policies**: Permissions in JSON format - like sudoers rules

**Why important**:
Basic security. Without properly configured IAM, you either block everything or expose everything.

**Mental model (Linux analogy)**:
```
IAM User   ≈ Linux User     (permanent identity)
IAM Group  ≈ Linux Group    (group permissions)
IAM Role   ≈ sudo -u / su   (temporary identity assumption)
IAM Policy ≈ sudoers rules  (what actions are allowed)

# Example sudoers (like IAM Policy):
arthur ALL=(ALL:ALL) ALL
# user   hosts=(runas_user:runas_group) commands

# Execute as another user:
sudo -u www-data whoami  # Like assuming a Role
```

**AWS example**:
```
User "arthur"
  └── in Group "Developers"
        └── attached Policy "EC2FullAccess"

Role "EC2-S3-Access"
  └── attached Policy "S3FullAccess"
  └── trusted by: EC2 service  # EC2 can "assume" this role
```

**Docs**: [IAM Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)

#### Concrete IAM Role Example

**Scenario**: Your EC2 needs to upload files to S3 (e.g., backups, logs).

**❌ Bad approach - hardcode credentials**:
```bash
# On the EC2, store AWS keys in ~/.aws/credentials
# Problems:
# - Keys can leak (git, logs, stolen)
# - Keys don't rotate automatically
# - If EC2 is compromised, keys are exposed forever
```

**✅ Good approach - use IAM Role**:
```hcl
# 1. Create a Role that EC2 can assume
resource "aws_iam_role" "ec2_s3_role" {
  name = "ec2-s3-backup-role"

  # Trust policy: WHO can assume this role
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"  # EC2 service can assume this role
      }
    }]
  })
}

# 2. Attach permissions: WHAT the role can do
resource "aws_iam_role_policy" "s3_access" {
  name = "s3-backup-access"
  role = aws_iam_role.ec2_s3_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:PutObject", "s3:GetObject"]
      Resource = "arn:aws:s3:::my-backup-bucket/*"
    }]
  })
}

# 3. Create instance profile (wrapper for EC2)
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-s3-profile"
  role = aws_iam_role.ec2_s3_role.name
}

# 4. Attach to EC2
resource "aws_instance" "web" {
  ami                  = "ami-xxx"
  instance_type        = "t3.micro"
  iam_instance_profile = aws_iam_instance_profile.ec2_profile.name
}
```

**Result**: EC2 can now use `aws s3 cp` without any credentials configured - AWS handles it automatically via temporary tokens.

**When to use Roles**:
- EC2 needs to access S3, RDS, Secrets Manager, etc.
- Lambda needs to access other AWS services
- Cross-account access (account A accesses resources in account B)
- Any service-to-service communication

---

### 3. Security Groups

**My understanding**:
Security Groups are instance-level firewalls. They filter incoming (ingress/inbound) and outgoing (egress/outbound) traffic by port/protocol/IP.

**Traffic directions**:
- **Ingress / Inbound** = traffic coming IN to the instance (from Internet)
- **Egress / Outbound** = traffic going OUT from the instance (to Internet)

**Why important**:
First line of defense. Misconfigured = instance exposed or unreachable.

**Mental model**:
```
Internet
    │
    ▼ INGRESS (inbound)
┌─────────────────┐
│ Security Group  │ ← Filters here
│  - 22: SSH     │
│  - 80: HTTP    │
│  - 443: HTTPS  │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│   EC2 Instance  │
└─────────────────┘
    │
    ▼ EGRESS (outbound)
┌─────────────────┐
│ Security Group  │ ← Also filters here!
│  - 0.0.0.0/0   │   (for apt, docker pull, etc.)
└─────────────────┘
    │
    ▼
Internet
```

**Best practices**:
- Port 22 (SSH) → Restrict to your IP, NEVER 0.0.0.0/0
- Port 80 → Redirect to 443 at application level
- Port 443 → Open to public (HTTPS)
- Databases → NEVER exposed to Internet, only via internal SGs
- Egress → At minimum allow all for updates (deny = no Internet from instance)

**Docs**: [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)

---

### 4. Layered Security

**My understanding**:
AWS security is layered. Each layer filters differently:

```
Internet
    │
    ▼
┌─────────────┐
│  WAF/CDN    │  ← Application level (CloudFront, WAF)
└─────────────┘
    │
    ▼
┌─────────────┐
│    NACL     │  ← Subnet level (stateless)
└─────────────┘
    │
    ▼
┌─────────────┐
│ Sec. Group  │  ← Instance level (stateful)
└─────────────┘
    │
    ▼
┌─────────────┐
│ UFW/iptables│  ← OS level (inside EC2)
└─────────────┘
```

**Why important**:
Defense in depth. If one layer fails, the others still protect.

---

## 💻 Minimal Example

### Context
Create a Security Group for a web server.

### Code
```hcl
# Security Group for WordPress
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Security group for web server"

  # SSH - restricted to your IP (INGRESS = inbound)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["X.X.X.X/32"]  # Your IP only!
  }

  # HTTP (INGRESS = inbound)
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTPS (INGRESS = inbound)
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound - allow all (EGRESS = outbound, for apt, docker pull, etc.)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}
```

**Output**: Security Group created, ready to attach to an EC2.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Blocked egress = no Internet access from instance

**Symptom**: `apt update` timeout, `curl` doesn't work from EC2

**What I did wrong**:
```hcl
# ❌ Wrong - Forgot the egress rule
resource "aws_security_group" "sg" {
  ingress { ... }
  # No egress = everything blocked by default!
}
```

**Solution**:
```hcl
# ✅ Correct - Always add egress (at minimum for updates)
egress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
}
```

**Time wasted**: 1h
**Lesson**: Security Groups = deny by default. Think about both directions (in + out).

---

## 📊 Stats

```yaml
Total time: 10h (50% assisted / 50% autonomous)
Used in: [[projects/2025-01-aws-terraform-ansible/learnings]]
```

---

**Last update**: 2025-01-21
**Next review**: 2025-02-21
