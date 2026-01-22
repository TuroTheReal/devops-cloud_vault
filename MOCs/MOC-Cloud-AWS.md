# MOC - Cloud AWS

> **What**: Learning path for AWS cloud services - from basics to production workloads.
>
> **Who**: DevOps engineers and developers learning cloud infrastructure.
>
> **Why**: Deploy and manage scalable, secure applications on AWS.

---

## 🎯 Learning Objective

Master AWS core services to provision, secure, and manage cloud infrastructure. Start with compute basics (EC2, IAM, Security Groups), then expand to storage, networking, and managed services.

**Real-world use case**: Deploy a production-ready web application on AWS with proper security, networking, and scalability.

---

## 📚 Learning Path

### Phase 1: AWS Basics (10h total) ✅ Completed

**Goal**: Understand core AWS concepts and deploy your first EC2 instance securely.

1. **[[concepts/aws/aws-fundamentals]]** ⭐⭐⭐ (10h)
   - **Why first**: Foundation for everything else on AWS
   - **You'll learn**: EC2, IAM (Users/Roles/Policies), Security Groups, basic services overview

---

### Phase 2: Networking & Storage (planned)

**Goal**: Design secure VPCs and manage data with S3.

2. **[[concepts/aws/aws-vpc]]** ⭐⭐⭐ (planned)
   - **Why now**: Understand how AWS networking works before deploying complex architectures
   - **You'll learn**: Subnets, route tables, NAT gateways, VPC peering

3. **[[concepts/aws/aws-alb]]** ⭐⭐⭐ (planned)
   - **Why now**: Most used load-balancer service, used everywhere
   - **You'll learn**: load balancing, routing, target groups & health checks, TLS with AWS ACM

4. **[[concepts/aws/aws-s3]]** ⭐⭐ (planned)
   - **Why now**: Most common storage service, used everywhere
   - **You'll learn**: Buckets, policies, lifecycle rules, static hosting

---

### Phase 3: Databases & Managed Services (planned)

**Goal**: Use managed services instead of self-hosting.

5. **[[concepts/aws/aws-rds]]** ⭐⭐⭐ (planned)
   - **Why now**: Managed databases are production standard
   - **You'll learn**: RDS setup, backups, Multi-AZ, security

6. **[[concepts/aws/aws-secrets-manager]]** ⭐⭐ (planned)
   - **Why now**: Secure credential management
   - **You'll learn**: Secrets rotation, integration with EC2/Lambda

---

### Phase 4: Containers & Serverless (planned)

**Goal**: Modern deployment patterns on AWS.

7. **[[concepts/aws/aws-ecs_eks]]** ⭐⭐⭐⭐ (planned)
   - **Why now**: Container orchestration on AWS
   - **You'll learn**: ECS Fargate, task definitions, service discovery

8. **[[concepts/aws/aws-lambda]]** ⭐⭐⭐ (planned)
   - **Why now**: Serverless for event-driven workloads
   - **You'll learn**: Functions, triggers, API Gateway integration

---

## 🛠️ Practice Project

**Apply your knowledge**: [[projects/2025-01-aws-terraform-ansible/learnings]]

**What you'll build**: WordPress infrastructure on EC2 with Terraform provisioning and Ansible configuration.

**Why this project**: Combines AWS basics (EC2, SG, IAM) with IaC tools in a real deployment scenario.

---

## 📊 Progress Tracking

```yaml
Total estimated time: 10h completed / ~50h planned
Difficulty: ⭐⭐⭐ (3/5)
Prerequisites: [[MOCs/MOC-Networking-Fundamentals]], [[MOCs/MOC-Linux-Security]]
```

| Phase | Status | Time |
|-------|--------|------|
| Phase 1: AWS Basics | ✅ Completed | 10h |
| Phase 2: Networking & Storage | ⏳ Planned | ~15h |
| Phase 3: Databases & Managed | ⏳ Planned | ~15h |
| Phase 4: Containers & Serverless | ⏳ Planned | ~20h |

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **AWS Certifications**: Solutions Architect Associate

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **IAM Roles > Access Keys**: Use roles for service-to-service communication, never hardcode credentials
2. **Security Groups**: Deny by default, allow explicitly. Don't forget egress rules!
3. **Least Privilege**: Grant minimum permissions needed

**Common pitfalls**:
- Forgetting egress rules in Security Groups (no internet access)
- Using root account instead of IAM users
- Hardcoding credentials instead of using IAM roles
- Public S3 buckets exposing data

**Cheatsheet**: [[cheatsheets/aws/aws-cli]]

---

**Created**: 2025-01-22
**Last updated**: 2025-01-22
