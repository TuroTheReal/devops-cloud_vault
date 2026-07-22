---
tags:
  - concept
  - aws
  - cloud
  - certification
  - clf-c02
  - status/learning
created: 2026-03-13
difficulty: 2
source: https://www.udemy.com/course/aws-certified-cloud-practitioner-new/
---

# AWS Cloud Practitioner CLF-C02 — Key Concepts Overview

**Related to**: [[concepts/aws/aws-fundamentals]]
**Official docs**: [AWS Docs](https://docs.aws.amazon.com/) | [AWS Certification](https://aws.amazon.com/certification/certified-cloud-practitioner/)
**Exam guide**: [CLF-C02 Official Exam Guide (PDF)](https://d1.awsstatic.com/training-and-certification/docs-cloud-practitioner/AWS-Certified-Cloud-Practitioner_Exam-Guide.pdf) | [Sample Questions](https://d1.awsstatic.com/training-and-certification/docs-cloud-practitioner/AWS-Certified-Cloud-Practitioner_Sample-Questions.pdf)

---

## ⚠️ Disclaimer

This is **NOT** a replacement for the actual training/course. It's a personal cheat sheet extracting key features and concepts for a **quick overview and understanding** of what the AWS Cloud Practitioner CLF-C02 certification covers — in my own words.

Based on [Stephane Maarek's Udemy course](https://www.udemy.com/course/aws-certified-cloud-practitioner-new/) and the [official CLF-C02 exam guide](https://d1.awsstatic.com/training-and-certification/docs-cloud-practitioner/AWS-Certified-Cloud-Practitioner_Exam-Guide.pdf). Content reflects the public exam blueprint — not actual exam questions.

---

## 🎯 TL;DR

AWS Cloud Practitioner = entry-level certification proving you understand **what** AWS services do and **when** to use them (not **how** to configure them in depth).

---

## 🤔 When to Use?

### ✅ Good for
- **CLF-C02 exam prep**: a fast, blueprint-aligned recap of what each service does and when to reach for it.
- **Cloud vocabulary**: mapping AWS service names to concepts when reading docs or talking to a team.

### ❌ Bad for
- **Hands-on depth / configuration** → see [[concepts/aws/aws-fundamentals]] and the official docs.

---

## 📚 Table of Contents

1. [Cloud Fundamentals](#1--cloud-fundamentals)
2. [IAM — Identity & Access Management](#2--iam--identity--access-management)
3. [EC2 — Compute & Storage](#3--ec2--compute--storage)
4. [ELB & ASG — Load Balancing & Scaling](#4--elb--asg--load-balancing--scaling)
5. [S3 — Storage](#5--s3--storage)
6. [Databases & Analytics](#6--databases--analytics)
7. [Other Compute Services](#7--other-compute-services)
8. [Deployment & Infrastructure Management](#8--deployment--infrastructure-management)
9. [Global Infrastructure](#9--global-infrastructure)
10. [Cloud Integration](#10--cloud-integration)
11. [Cloud Monitoring](#11--cloud-monitoring)
12. [VPC & Networking](#12--vpc--networking)
13. [Security & Compliance](#13--security--compliance)
14. [Machine Learning](#14--machine-learning)
15. [Account Management, Billing & Support](#15--account-management-billing--support)
16. [Advanced Identity](#16--advanced-identity)
17. [Other Services](#17--other-services)
18. [Well-Architected Framework & Ecosystem](#18--well-architected-framework--ecosystem)

---

## 1. 🌐 Cloud Fundamentals

### What is Cloud Computing?

**Pay-as-you-go** model — use what you need, pay for what you use.

### Cloud Types

| Type | Description | Use case |
|------|-------------|----------|
| **Public** | Infrastructure owned by providers (AWS, Azure, GCP) | Most workloads |
| **Private** | Dedicated infrastructure for one organization | Sensitive data, compliance |
| **Hybrid** | Mix of public + private | Gradual migration, specific compliance needs |

### Cloud Advantages

| Advantage | Description |
|-----------|-------------|
| **Cost-effectiveness** | Pay-as-you-go, no upfront hardware |
| **Flexibility** | On-demand, change when needed |
| **Scalability** | Handle larger loads (spikes/bursts) |
| **Elasticity** | Scale out/in automatically |
| **High Availability** | Multiple datacenters |
| **Agility** | Rapidly develop, test, launch |

### Service Models

```
YOU MANAGE ◄──────────────────────────────────► AWS MANAGES

On-Premise         IaaS              PaaS            SaaS
┌──────────┐   ┌──────────┐     ┌──────────┐    ┌──────────┐
│ App      │   │ App      │     │ App      │    │          │
│ Data     │   │ Data     │     │ Data     │    │          │
│ Runtime  │   │ Runtime  │     │──────────│    │  Just    │
│ OS       │   │ OS       │     │          │    │  use it  │
│ Network  │   │──────────│     │          │    │          │
│ Storage  │   │          │     │          │    │          │
│ Servers  │   │          │     │          │    │          │
└──────────┘   └──────────┘     └──────────┘    └──────────┘
 You do ALL     You pick the     You focus on     You just
                tools/infra      code & data      pay & use
```

### AWS Global Infrastructure

- **Region** = cluster of data centers (choose based on: proximity, compliance, pricing, latency, available services)
- **Availability Zone (AZ)** = isolated datacenter within a region (3 to 6 per region, connected with ultra-low latency networking)

### Shared Responsibility Model

```
┌─────────────────────────────────────────────┐
│              YOU (Customer)                  │
│  Security IN the cloud:                     │
│  Data, IAM, OS, SG, encryption, firewall,   │
│  network config, app, platform              │
├─────────────────────────────────────────────┤
│              AWS                            │
│  Security OF the cloud:                     │
│  Hardware, compute, storage, database,      │
│  networking, regions, software              │
└─────────────────────────────────────────────┘
```

### 3 Fundamentals of AWS Pricing

**Compute** + **Storage** + **Data transfer OUT** of the cloud

> CAPEX = Capital Expenditure (upfront investment)
> OPEX = Operational Expenditure (ongoing costs) ← Cloud model

---

## 2. 🔐 IAM — Identity & Access Management

> Like Linux users/groups system but for AWS.

### Core Concepts

```
Root Account (DO NOT USE for daily tasks)
│
├── Group: Developers
│   ├── User: Alice ──► Policy: EC2FullAccess
│   └── User: Bob   ──► Policy: S3ReadOnly
│
├── Group: Admins
│   └── User: Carol ──► Policy: AdministratorAccess
│
└── Role: EC2-S3-Access ──► Used BY AWS services (not humans)
```

- **Users** can belong to multiple groups
- **Groups** contain only users (no nesting)
- **Policies** = JSON documents defining permissions
- **Roles** = identities for AWS services (EC2 accessing S3, Lambda accessing DynamoDB...)

### IAM Policy Structure

```json
{
  "Version": "2012-10-17",        // Always this date
  "Statement": [{
    "Sid": "AllowS3Read",          // Optional: statement ID
    "Effect": "Allow",             // Allow or Deny
    "Principal": {"AWS": "arn..."}, // Who this applies to
    "Action": ["s3:GetObject"],    // What actions
    "Resource": ["arn:aws:s3:::*"], // On which resources
    "Condition": {}                // Optional: when in effect
  }]
}
```

### Best Practices

| Practice | Why |
|----------|-----|
| Never use root except for initial setup | Root = unlimited power, can't be restricted |
| 1 physical person = 1 IAM user | Traceability |
| MFA on all users, **mandatory** on root | Protect against credential theft |
| Use Roles for services, never hardcode keys | Keys can leak, roles auto-rotate |
| Least privilege principle | Only grant what's needed |
| Audit credentials regularly | Detect unused/compromised keys |
| Never share IAM users or access keys | Accountability |

### CLI Access

```bash
aws iam list-users  # List all IAM users
```

> **Never configure AWS access keys directly on EC2** — attach an IAM Role instead.

---

## 3. 💻 EC2 — Compute & Storage

### EC2 = Elastic Compute Cloud

**User Data** = bootstrap script that runs once at first launch (needs `sudo`).

### Instance Types

| Type | Optimized for | Prefix | Use case |
|------|--------------|--------|----------|
| **General Purpose** | Balanced | `t`, `m` | Web servers, small DBs |
| **Compute Optimized** | High CPU | `c` | Batch processing, ML, gaming servers |
| **Memory Optimized** | High RAM | `r`, `x` | In-memory DBs, real-time processing |
| **Storage Optimized** | High I/O | `i`, `d` | Data warehousing, distributed file systems |

### Security Groups

```
Internet ──► Security Group (firewall) ──► EC2 Instance
                │
                ├── Inbound rules (ingress): who can come IN
                └── Outbound rules (egress): what can go OUT
```

- ALLOW rules only (no DENY)
- Can reference other Security Groups (no need to hardcode IPs)
- **Timeout = often a Security Group issue**
- Stateful: if inbound allowed, return traffic is auto-allowed

### Purchase Options

| Option | Analogy (hotel) | Discount | Commitment | Best for |
|--------|----------------|----------|------------|----------|
| **On-Demand** | Walk-in, full price | 0% | None | Short/unpredictable workloads |
| **Reserved** | Book ahead, long stay discount | ~40-70% | 1-3 years | Steady-state workloads |
| **Savings Plan** | Pay X$/hour, use any room | ~60-66% | 1-3 years | Flexible commitment |
| **Spot** | Empty room, may get kicked out | Up to 90% | None | Fault-tolerant, flexible |
| **Dedicated Host** | Rent the whole hotel | Premium | Optional | Licensing, compliance |
| **Dedicated Instance** | Private room, basic | Premium | Optional | Isolation requirements |
| **Capacity Reservation** | Book room even if you don't come | 0% | Any duration | Guaranteed capacity in specific AZ |

### EC2 Storage

```
┌─────────────────────────────────────────────────────────────┐
│                      EC2 Instance                           │
│                                                             │
│  ┌─────────────────┐   ┌──────────────────────────────┐     │
│  │ Instance Store   │   │ EBS Volume (attached)        │     │
│  │ - Ephemeral      │   │ - Persistent (survives stop) │     │
│  │ - Best I/O perf  │   │ - Locked to 1 AZ            │     │
│  │ - Lost on stop   │   │ - Snapshot to move/backup    │     │
│  │ - Good for cache │   │ - Can resize                 │     │
│  └─────────────────┘   └──────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘

                         ┌──────────────────────────────┐
                         │ EFS (Network File System)     │
                         │ - Mount on 100s of EC2        │
                         │ - Multi-AZ                    │
                         │ - Linux only                  │
                         │ - Pay per use                 │
                         │ - EFS-IA: 92% cheaper for     │
                         │   infrequent access            │
                         └──────────────────────────────┘
```

#### EBS = Elastic Block Store

- Persists after instance termination
- Locked to 1 AZ — use **snapshots** to move across AZ/region
- Can attach/detach, resize
- **io1/io2 volumes CAN be multi-attached** (same AZ only)

#### EBS Snapshots

- Copy data across AZ/regions
- **Snapshot Archive**: 75% cheaper, 24-72h to restore
- **Recycle Bin**: deleted snapshots go to bin (retention: 1 day to 1 year)
- Best practice: detach volume before snapshotting

#### AMI = Amazon Machine Image

- Custom image: your software, config, OS
- Region-specific (can be copied across regions)
- Create from EC2 → deploy new EC2 from image

#### EC2 Image Builder

- Automate creation, test, validate, distribute AMIs
- Multi-AZ capable

#### Amazon FSx

| Type | What | Use case |
|------|------|----------|
| **FSx for Windows** | Windows-native file system (SMB/NTFS) | Windows workloads |
| **FSx for Lustre** | High-perf Linux cluster storage | ML, analytics, video, HPC (up to 100 GB/s) |

### Scaling Concepts

```
Vertical Scaling          Horizontal Scaling
(Scale UP/DOWN)           (Scale OUT/IN)

┌───────────┐             ┌───┐ ┌───┐ ┌───┐
│           │             │   │ │   │ │   │
│  BIGGER   │             │ S │ │ S │ │ S │
│  SERVER   │             │   │ │   │ │   │
│           │             └───┘ └───┘ └───┘
└───────────┘              More small nodes

Higher specs              More instances
```

> **High Availability** = at least 2 AZs

---

## 4. ⚖️ ELB & ASG — Load Balancing & Scaling

### ELB = Elastic Load Balancer

```
         Users
           │
           ▼
    ┌──────────────┐
    │     ELB      │  ← Single DNS entry
    │  (1 endpoint)│  ← Health checks on instances
    └──────┬───────┘  ← SSL termination (HTTPS)
       ┌───┼───┐
       ▼   ▼   ▼
     EC2  EC2  EC2    ← Target Group
```

| Type | Layer | Protocol | Key feature |
|------|-------|----------|-------------|
| **ALB** (Application) | 7 | HTTP/HTTPS, gRPC | URL-based routing, static DNS |
| **NLB** (Network) | 4 | TCP/UDP | Ultra-high perf, static IP (Elastic IP) |
| **GWLB** (Gateway) | 3 | GENEVE | Route to firewall/IDS on EC2 |
| ~~CLB~~ (Classic) | 4+7 | — | Retired |

### ASG = Auto Scaling Groups

- Scale in/out EC2 instances automatically
- Manages instance count in a **Target Group**
- Replaces unhealthy instances
- Optimizes cost (no over-provisioning)

| Strategy | How it works |
|----------|-------------|
| **Manual** | You set desired count by hand |
| **Dynamic (Simple/Step)** | If CPU > X%, add N instances |
| **Target Tracking** | "I want average CPU at 40%" |
| **Scheduled** | "Add 5 instances at 9am weekdays" |
| **Predictive** | ML analyzes past patterns, scales ahead |

---

## 5. 📦 S3 — Storage

### S3 = Simple Storage Service

- Object storage organized in **Buckets**
- Bucket name must be **globally unique**
- Defined at **region level**
- Objects stored by **key** (= full path)
- Objects can have: metadata, version ID, tags

Use cases: backup, disaster recovery, archive, static website hosting, application/media hosting, analytics, software delivery.

### S3 Security

```
┌─────────────────────────────────────────────┐
│              S3 Bucket                       │
│                                              │
│  User-Based:                                 │
│  └── IAM Policies (what API calls allowed)   │
│                                              │
│  Resource-Based:                             │
│  ├── Bucket Policies (cross-account, public) │
│  ├── Object ACL (fine grain, can disable)    │
│  └── Bucket ACL (less common, can disable)   │
└─────────────────────────────────────────────┘
```

Bucket policies: good for granting public access, enforcing encryption at upload, cross-account access.

> For static websites: everything in the S3 bucket must be **publicly readable**.

### S3 Features

| Feature | What it does |
|---------|-------------|
| **Versioning** | Same key overwrite = new version. Protects against unintended deletes, allows rollback. Null by default |
| **Replication (CRR)** | Cross-Region: compliance, low latency. Async, needs IAM permissions |
| **Replication (SRR)** | Same-Region: log aggregation, env replication |
| **Encryption** | SSE (server-side, default) or CSE (client-side) |
| **IAM Access Analyzer** | Evaluate S3 policies and access permissions |

### S3 Storage Classes

```
HOT ◄────────────────────────────────────────────► COLD

Standard    Standard-IA    One Zone-IA    Glacier       Glacier      Glacier Deep
                                          Instant       Flexible     Archive
                                          Retrieval     Retrieval

Frequent    Infrequent     Infrequent     Archive       Archive      Long-term
access      access         access         (ms access)   (min-hours)  archive
            (cheap)        (cheapest,                                 (12-48h)
                           1 AZ only)

$0.023/GB   $0.0125/GB     $0.01/GB       $0.004/GB     $0.0036/GB   $0.00099/GB
```

- **Intelligent Tiering**: auto-moves objects between tiers based on usage (small fee)
- **S3 Express One Zone**: Directory Bucket in single AZ, highest perf, cheaper than Standard
- Can switch between classes manually or with lifecycle rules

### Shared Responsibility for S3

| You | AWS |
|-----|-----|
| Versioning, bucket policies, replication setup | Infrastructure, replication reliability |
| Logging, monitoring, encryption config | Configuration analysis, compliance validation |
| Storage class selection, backups | Hardware, data center security |

---

## 6. 🗄️ Databases & Analytics

### OLTP vs OLAP

```
OLTP (Transactional)              OLAP (Analytical)
─────────────────────             ─────────────────────
Simple, fast queries              Complex, slow queries
Current data                      Historical data
Thousands of users                Dozens of analysts
Frequent writes                   Rare writes
Example: buy a product            Example: sales last 5 years
Services: RDS, Aurora,            Services: Redshift, Athena,
          DynamoDB                           EMR, QuickSight
```

### Quick Recap

| Service | Type | What | Key feature |
|---------|------|------|-------------|
| **RDS** | SQL (OLTP) | Managed MySQL/PostgreSQL/Oracle/SQL Server | Multi-AZ, auto backup, no SSH access |
| **Aurora** | SQL (OLTP) | AWS-built RDS++ | 3-5x faster, auto-grow 10GB→256TB, serverless option |
| **ElastiCache** | Cache | Managed Redis/Memcached | In-memory, ms latency, reduce DB load |
| **DynamoDB** | NoSQL | Serverless key-value store | Single-digit ms, multi-AZ, pay per request |
| **DAX** | Cache | DynamoDB Accelerator | Microsecond latency cache for DynamoDB |
| **DynamoDB Global Tables** | NoSQL | Multi-region DynamoDB | Active-active replication everywhere |
| **DocumentDB** | NoSQL | MongoDB compatible (Aurora for Mongo) | Fully managed, HA, auto-grow |
| **Neptune** | Graph | Graph database | Social networks, fraud detection, up to 15 replicas |
| **Timestream** | Time-series | TSDB | 1000x faster than relational for time data, IoT/metrics |
| **Redshift** | OLAP | Data warehouse (PostgreSQL-based) | Columnar storage, MPP, 10x faster, serverless option |
| **Athena** | Query | Serverless SQL on S3 | $5/TB scanned, no data import needed |
| **EMR** | Big Data | Managed Hadoop/Spark | Auto-scaling clusters, ML, data processing |
| **QuickSight** | BI | Interactive dashboards | Serverless, integrates with Aurora/Redshift/RDS/S3 |
| **Managed Blockchain** | Blockchain | Managed blockchain networks | Ethereum or Hyperledger Fabric |
| **Glue** | ETL | Extract, Transform, Load | Serverless, prepare data for analytics |
| **DMS** | Migration | Database Migration Service | Zero downtime, homogeneous + heterogeneous migration |

---

## 7. 🐳 Other Compute Services

### Container & Serverless Landscape

```
                    ┌──────────────────────────────────────┐
                    │         Container Services            │
                    │                                       │
  You manage EC2    │   ECS ◄──── ECR (image registry)      │
                    │    │                                   │
  AWS manages all   │   Fargate ◄── ECR                     │
                    │                                       │
  Kubernetes        │   EKS (Fargate or EC2)                │
                    └──────────────────────────────────────┘

                    ┌──────────────────────────────────────┐
                    │        Serverless Services            │
                    │                                       │
                    │   Lambda ──► API Gateway               │
                    │   (functions)  (expose as REST API)    │
                    │                                       │
                    │   Batch (start/end jobs, no time limit)│
                    │                                       │
                    │   Lightsail (simple VPS alternative)   │
                    └──────────────────────────────────────┘
```

### Quick Recap

| Service | Type | What | Key feature |
|---------|------|------|-------------|
| **ECS** | Container | Docker on AWS (you manage EC2) | ALB integration, you provision infrastructure |
| **Fargate** | Serverless Container | Docker without managing servers | Just specify CPU/RAM, AWS handles the rest |
| **ECR** | Registry | Private Docker Hub for AWS | Store images, used by ECS/Fargate |
| **EKS** | Kubernetes | Managed K8s | Standard K8s compatible, Fargate or EC2 |
| **Lambda** | Serverless Function | Event-driven code execution | 15min max, pay per request, up to 10GB, multi-language |
| **API Gateway** | API Management | Create/manage REST APIs | Serverless, scalable, auth, pairs well with Lambda |
| **Batch** | Batch Processing | Large-scale batch jobs (start/end) | Docker on ECS, no time limit, cost-optimized |
| **Lightsail** | Simple Compute | Simple VPS | Server + storage + DB + networking, simplified |

> **Serverless** = you don't manage/see/provision servers. There IS a server, you just don't deal with it.

---

## 8. 🚀 Deployment & Infrastructure Management

### Quick Recap

| Service | Type | What | Key feature |
|---------|------|------|-------------|
| **CloudFormation** | IaC | AWS Terraform equivalent | YAML/JSON templates, automated diagrams, cost planning |
| **CDK** | IaC (dev-friendly) | CloudFormation in Python/TypeScript/Java | Code → compiles to CloudFormation templates |
| **Beanstalk** | PaaS | Managed deployment platform | Just upload code. 3 models: Single (dev), LB+ASG (prod), ASG only (workers) |
| **CodeDeploy** | Deployment | Deploy/upgrade apps to servers | Hybrid (on-premise + EC2) |
| **CodeCommit** | Source Control | AWS GitHub equivalent | Private Git repos with IAM integration |
| **CodeBuild** | Build/Test | Compile and test code | Serverless, pay per build |
| **CodePipeline** | CI/CD | Full pipeline orchestration | Source → Build → Test → Deploy |
| **CodeArtifact** | Artifact Repo | Store dependencies (npm, pip, Maven) | Secure package management |
| **SSM** (Systems Manager) | Management | Manage servers at scale | Hybrid, auto patching, run commands across all servers, needs SSM agent |
| **SSM Session Manager** | Secure Shell | SSH without port 22 | No SSH key needed, logs to S3/CloudWatch |
| **SSM Parameter Store** | Secrets/Config | Store config, secrets, API keys | Serverless, IAM permissions, optional encryption + versioning |

### CI/CD Pipeline Flow

```
CodeCommit ──► CodeBuild ──► CodeDeploy
(source)       (build/test)   (deploy)
     └──────────────┴──────────────┘
              CodePipeline
           (orchestrates all)
```

---

## 9. 🌍 Global Infrastructure

### Why Go Global?

Lower latency, disaster recovery (failover), attack protection.

### Route 53 — Managed DNS

| Routing Policy | Health Check | How it works | Use case |
|---------------|-------------|--------------|----------|
| **Simple** | No | Returns IP(s) | Single resource |
| **Weighted** | Optional | X% traffic to each endpoint | A/B testing, gradual deploy |
| **Latency** | Optional | Route to lowest latency | Global apps, best UX |
| **Failover** | Required | Primary → Secondary if HC fails | HA, disaster recovery |

### CDN & Acceleration

```
User (far away)
     │
     ▼
┌─────────────┐     ┌─────────────┐
│ CloudFront  │────►│  S3 / EC2   │   CloudFront = CDN, caches at edge
│ (Edge)      │     │  (Origin)   │   + DDoS protection + WAF
└─────────────┘     └─────────────┘

┌──────────────────┐
│ S3 Transfer      │  Upload via edge → S3 bucket (faster for far locations)
│ Acceleration     │
└──────────────────┘

┌──────────────────┐
│ Global           │  AWS backbone network (no caching), 60% faster
│ Accelerator      │  Direct to edge → AWS private network → app
└──────────────────┘
```

### Edge & On-Premise

| Service | What | Use case |
|---------|------|----------|
| **Outposts** | Physical AWS rack in YOUR datacenter | Low latency, data residency, hybrid |
| **Wavelength** | Deploy in 5G operator datacenter | Ultra-low latency (<10ms): autonomous cars, VR, gaming |
| **Local Zones** | Mini-region near cities | Lower latency for specific metro areas |

### Global Architecture Patterns

```
Availability ──────────────────────────────────► Higher
Complexity ────────────────────────────────────► Higher

Single Region     Single Region     Multi-Region       Multi-Region
Single AZ         Multi-AZ          Active/Passive     Active/Active

┌─────┐           ┌─────┐ ┌─────┐  ┌─────┐ ┌─────┐   ┌─────┐ ┌─────┐
│ AZ1 │           │ AZ1 │ │ AZ2 │  │R/W  │ │Read │   │R/W  │ │R/W  │
└─────┘           └─────┘ └─────┘  │Reg1 │ │Reg2 │   │Reg1 │ │Reg2 │
                                   └─────┘ └─────┘   └─────┘ └─────┘
No HA             HA within        HA + read          HA + low latency
No low latency    region           globally           globally (hard)
```

---

## 10. 🔗 Cloud Integration

Communication between services can be **synchronous** (direct) or **asynchronous** (via queue/topic).

```
Synchronous:   App A ──────► App B        (direct call, coupled)

Asynchronous:  App A ──► SQS Queue ──► App B   (decoupled)
               App A ──► SNS Topic ──► App B + App C + App D   (fan-out)
```

### Quick Recap

| Service | Type | What | Key feature |
|---------|------|------|-------------|
| **SQS** | Queue | Simple Queue Service (oldest AWS service) | Decoupling, 1-10K msg/sec, 4-14 days retention, FIFO option, messages deleted after read |
| **SNS** | Pub/Sub | Simple Notification Service | 1 message → many subscribers (email, SMS, Lambda, SQS...), up to 12.5M sub/topic |
| **Kinesis Data Streams** | Streaming | Real-time data streaming | Collect, process, analyze streaming data at any scale |
| **Amazon MQ** | Message Broker | Managed RabbitMQ / ActiveMQ | For migration to AWS (existing apps using these brokers), doesn't scale like SQS/SNS |

---

## 11. 📊 Cloud Monitoring

### Quick Recap

| Service | Type | What | Key feature |
|---------|------|------|-------------|
| **CloudWatch Metrics** | Monitoring | Collect metrics from AWS services | EC2 (CPU, status), EBS, S3, Billing (us-east-1 only) |
| **CloudWatch Alarms** | Alerting | Trigger actions on metric thresholds | Auto-scaling, SNS notifications, EC2 actions |
| **CloudWatch Logs** | Logging | Centralized log collection | Needs agent + IAM, real-time, configurable retention, hybrid |
| **EventBridge** | Event Bus | Schedule or react to AWS events | Cron → Lambda, EC2 launch → notify, replay archived events, partner events |
| **CloudTrail** | Audit | Track ALL API calls on your AWS account | Enabled by default, logs to S3/CloudWatch, governance |
| **X-Ray** | Tracing | Distributed tracing and debugging | Service graph, find bottlenecks, SLA checking |
| **CodeGuru** | Code Analysis | ML-powered code review | Finds vulnerabilities, best practices, input validation issues |
| **Health Dashboard** | Status | Service health + your account-specific issues | Global service status + personalized alerts for YOUR resources |

### Who Did What? — Choosing the Right Tool

```
"WHO did WHAT?"          → CloudTrail  (API audit trail)
"HOW are resources set?" → Config      (configuration snapshots)
"WHAT threats detected?" → GuardDuty   (continuous threat monitoring)
```

---

## 12. 🌐 VPC & Networking

### VPC Architecture

```
┌─────────────────────────── VPC ──────────────────────────┐
│                                                           │
│  ┌──────────────── Public Subnet ─────────────────┐       │
│  │                                                 │       │
│  │  EC2 (web) ◄── Security Group                   │       │
│  │     │                                           │       │
│  │     │          NAT Gateway                      │       │
│  └─────┼──────────────┬────────────────────────────┘       │
│        │              │                                    │
│  ┌─────┼──────────────┼── Private Subnet ──────────┐       │
│  │     │              │                             │       │
│  │  EC2 (app) ◄── Security Group                    │       │
│  │     │              │                             │       │
│  │  RDS (db)          │                             │       │
│  │                    │                             │       │
│  └────────────────────┼─────────────────────────────┘       │
│                       │                                    │
│              Internet Gateway (IGW)                        │
└───────────────────────┼────────────────────────────────────┘
                        │
                    Internet
```

**Traffic flow for private subnet → internet**:
Private EC2 → NAT Gateway (in public subnet) → IGW → Internet

### Security Group vs NACL

| Aspect | Security Group | NACL |
|--------|---------------|------|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful (return auto-allowed) | Stateless (must allow both directions) |
| **Rules** | ALLOW only | ALLOW + DENY |
| **Rule order** | All rules evaluated | Numbered order (lowest first) |
| **Source** | IP or Security Group | IP only |
| **Default** | Deny all in, allow all out | Allow all in/out |

### Connectivity Options

```
VPC-to-VPC:
  VPC Peering ──── Direct connection (NOT transitive: A↔B + B↔C ≠ A↔C)
  PrivateLink ──── Expose service between VPCs (no peering needed)
  Transit GW ──── Central hub for all connections

VPC-to-Internet:
  IGW ──────────── Public subnet internet access
  NAT GW ──────── Private subnet outbound only

VPC-to-On-Premise:
  Site-to-Site VPN ── Over internet, encrypted (minutes to setup, cheap)
  Direct Connect ──── Physical fiber (1 month setup, 1-100 Gbps, expensive)
  Client VPN ──────── OpenVPN for individual users to access private resources

AWS-to-AWS:
  VPC Endpoints ──── Access AWS services (S3, DynamoDB) via private network
```

| | Site-to-Site VPN | Direct Connect |
|---|---|---|
| **Connection** | Internet (encrypted) | Physical fiber |
| **Setup** | Minutes | ~1 month |
| **Speed** | Limited by ISP | 1-100 Gbps |
| **Latency** | Variable | Very low, consistent |
| **Cost** | Cheap | Expensive |
| **Best for** | Dev/test, backup | Production, critical workloads |

- **Flow Logs**: capture IP traffic logs (VPC/subnet/ENI level)

---

## 13. 🛡️ Security & Compliance

### Protection Layers

```
Internet
    │
    ▼
┌──────────┐
│ Shield   │  Layer 3/4: DDoS protection (Standard = free, Advanced = $3K/month)
└──────────┘
    │
    ▼
┌──────────┐
│ WAF      │  Layer 7: SQL injection, XSS, HTTP floods (on ALB/API GW/CloudFront)
└──────────┘
    │
    ▼
┌──────────┐
│ Network  │  Layer 3-7: Protect entire VPC, any direction
│ Firewall │
└──────────┘
    │
    ▼
┌──────────┐
│ SG/NACL  │  Instance/subnet level firewall
└──────────┘

Firewall Manager = manage ALL rules across your entire AWS Organization
```

### Encryption & Key Management

| | KMS | CloudHSM |
|---|---|---|
| **Hardware** | Shared (multi-tenant) | Dedicated hardware |
| **Key control** | AWS manages | You manage |
| **Can export keys** | No | Yes |
| **Compliance** | FIPS 140-2 Level 2 | FIPS 140-2 Level 3 |
| **Cost** | $1/key/month + usage | ~$1-3/hour/HSM |
| **Use case** | Most workloads | Strict compliance |

```
Data at Rest ── stored on disk/DB ── Server-side encryption (SSE) ── S3, EBS, RDS
Data in Transit ── moving over network ── TLS/SSL ── HTTPS, VPN
```

- **ACM** (Certificate Manager): provision/manage SSL/TLS certs, free for public, auto-renewal
- **Secrets Manager**: store/rotate secrets, encrypted with KMS, RDS integration

### Security Services

| Service | What | Detects/Monitors |
|---------|------|-----------------|
| **GuardDuty** | ML threat detection | Malicious activity, crypto mining (analyzes CloudTrail/Flow Logs/DNS) |
| **Inspector** | Vulnerability scanning | CVEs on EC2/ECR/Lambda, network reachability |
| **Config** | Configuration auditing | Config drift, compliance over time (SG rules, bucket access...) |
| **Macie** | Data privacy | PII in S3 (SSN, credit cards...) via ML/pattern matching |
| **Security Hub** | Central dashboard | Aggregates findings from GuardDuty/Inspector/Macie, requires Config |
| **Detective** | Root cause analysis | Investigate WHY (ML/graphs on VPC Flow/CloudTrail/GuardDuty) |
| **Artifact** | Compliance reports | Download SOC/PCI reports, manage agreements (BAA, HIPAA) |
| **IAM Access Analyzer** | External access audit | Resources shared outside your zone of trust |
| **Abuse** | Report abuse | Report illegal/abusive activity from AWS IPs |

### Root User — Exclusive Privileges

Actions that **cannot** be delegated to IAM users:
- Change account settings (name, email, password)
- Close AWS account
- Change/cancel support plan
- Register for GovCloud
- Sign up as Reserved Instance Marketplace seller

> **Penetration testing** is allowed on AWS, but some attack types are prohibited (they may think you're attacking their infrastructure).

---

## 14. 🤖 Machine Learning

### Quick Recap

| Service | What | Use case |
|---------|------|----------|
| **Rekognition** | Image/video analysis | Object/face detection, content moderation |
| **Transcribe** | Speech → text | Subtitles, transcripts (auto PII removal, auto language ID) |
| **Polly** | Text → speech | Voice apps, audiobooks (opposite of Transcribe) |
| **Translate** | Language translation | Localization, 75+ languages |
| **Lex** | Chatbot builder | Same tech as Alexa, ASR + NLU |
| **Connect** | Cloud call center | Virtual contact center, CRM integration |
| **Comprehend** | NLP | Sentiment analysis, entity extraction, topic modeling |
| **SageMaker** | ML platform | Build/train/deploy custom ML models end-to-end |
| **Kendra** | Document search | Intelligent search, extract answers from docs |
| **Personalize** | Recommendations | Real-time personalization (same tech as Amazon.com) |
| **Textract** | OCR | Extract text/data from scanned documents, forms, handwriting |

---

## 15. 💰 Account Management, Billing & Support

### AWS Organizations

```
Management Account (master) ──── Pays everything (consolidated billing)
│
├── OU: Production
│   ├── Account: Prod-App
│   └── Account: Prod-DB
│
├── OU: Development
│   ├── Account: Dev-Team1
│   └── Account: Dev-Team2
│
└── OU: Security
    └── Account: Audit

SCP (Service Control Policies) applied at OU or Account level
├── Whitelist/blacklist IAM actions
├── NOT applied on Management Account
├── Applied on everything else INCLUDING root users in child accounts
└── Must have explicit ALLOW
```

- **Combined usage** → volume pricing discounts
- **Control Tower**: automate/govern multi-account setup
- **RAM** (Resource Access Manager): share resources across accounts
- **Service Catalog**: approved product list for easy onboarding

### Pricing Models

| Model | Description |
|-------|-------------|
| **Pay-as-you-go** | Pay per second/hour/request |
| **Save when you reserve** | 1-3 year commitment = discount |
| **Pay less by using more** | Volume discounts |
| **Pay less as AWS grows** | AWS passes savings to customers |

### Savings Plans

```
Commit to $X/hour for 1-3 years → get discounted pricing

Usage ≤ commitment → Discounted rate
Usage > commitment → On-Demand rate

Payment: All Upfront (max discount) > Partial > No Upfront (least discount)
```

| Type | Scope | Discount | Flexibility |
|------|-------|----------|-------------|
| **Compute Savings Plan** | EC2 + Fargate + Lambda, any region | Up to ~66% | Most flexible |
| **EC2 Instance Savings Plan** | EC2 only, locked to 1 family + 1 region | Up to ~72% | Less flexible |
| **SageMaker Savings Plan** | SageMaker only, any region | Up to ~64% | SageMaker-specific |

> **Reserved Instances** = locked to instance type + region + OS. **Savings Plans** = commit $/hour, more flexible. AWS recommends Savings Plans for new commitments.

### Billing & Cost Tools

```
ESTIMATING          TRACKING              MONITORING
───────────         ────────              ──────────
Pricing             Cost Allocation       Billing Alarms
Calculator          Tags                  (CloudWatch)
                    │                         │
                    Cost & Usage          AWS Budgets
                    Reports               (more powerful,
                    │                     forecast + ceiling)
                    Cost Explorer             │
                    (12mo forecast)       Cost Anomaly
                                         Detection (ML)
```

- **Compute Optimizer**: ML-powered recommendations to reduce costs
- **Service Quotas**: monitor/request quota increases
- **Trusted Advisor**: high-level account assessment (cost, performance, security, fault tolerance, service limits) — needs Business or Enterprise plan for full checks

### Support Plans

| Plan | Price | Response (critical) | Key features |
|------|-------|-------------------|--------------|
| **Basic** | Free | — | 7 core Trusted Advisor checks, Health Dashboard |
| **Developer** | $29/mo | 12h general | 1 contact, business-hours email |
| **Business** | $100/mo | 1h (prod down) | Unlimited contacts, full Trusted Advisor, 24/7 phone |
| **Enterprise On-Ramp** | $5,500/mo | 30min (business-critical) | Pool of TAMs, Concierge, 1 IEM/year |
| **Enterprise** | $15,000/mo | 15min (business-critical) | Dedicated TAM, Concierge, IEM included |

---

## 16. 🆔 Advanced Identity

### Quick Recap

| Service | What | Use case |
|---------|------|----------|
| **STS** (Security Token Service) | Generate temporary credentials (15min-12h) | Cross-account access, federated users, MFA |
| **Cognito** | User auth for web/mobile apps | App login (NOT IAM users), social login, millions of users |
| **Directory Service** | Managed Microsoft Active Directory | Corporate credentials for AWS (WorkSpaces, RDS, EC2) |
| **IAM Identity Center** (ex-SSO) | Single Sign-On portal | One login → multiple AWS accounts + SaaS apps (Slack, Salesforce...) |

> **Cognito** = for your APP users. **IAM** = for your AWS account access. Never use IAM for app users.

---

## 17. 📎 Other Services

> Rarely on exam, but good to know the definition.

| Service | What | Use case |
|---------|------|----------|
| **WorkSpaces** | Managed Desktop as a Service (DaaS) | Remote desktops, deploy close to users for latency |
| **AppStream 2.0** | App streaming to browser | Stream apps without provisioning infrastructure |
| **IoT Core** | Connect IoT devices to AWS | Smart devices, sensors |
| **AppSync** | Sync data across apps (GraphQL) | Real-time mobile/web data sync |
| **Amplify** | Full-stack web/mobile dev platform | Rapid frontend + backend development |
| **Infrastructure Composer** | Visual serverless app designer | Drag-and-drop → generates CloudFormation |
| **Device Farm** | Test apps on real devices | Responsive testing, get reports/logs/screenshots |
| **AWS Backup** | Centralized backup management | Cross-region/account, PITR, scheduled |
| **DRS** (Elastic Disaster Recovery) | Recover servers into AWS | Protect critical databases, physical/virtual migration |
| **DataSync** | Move large data from on-premise | First full load, then incremental replication |
| **FIS** (Fault Injection Simulator) | Chaos engineering | Test how your infra handles failures |
| **Step Functions** | Serverless visual workflow | Orchestrate Lambda functions |
| **Ground Station** | Satellite communication | Download satellite data to VPC |
| **Pinpoint** | 2-way communication (SMS, email, voice) | Marketing campaigns, delivery schedules |

### Disaster Recovery Strategies

```
CHEAPEST ◄──────────────────────────────────────► FASTEST RECOVERY

Backup &         Pilot Light       Warm Standby      Multi-Site
Restore                                               (Hot)

Backup data      Core functions     Full app,         Full app,
Restore if       only (e.g., DB)    minimum size      full size
disaster

Hours            Minutes            Minutes           Real-time
                                                      failover
```

> Exam tip: cheapest = **Backup & Restore**

### Cloud Migration — 7Rs

| Strategy | What | Example |
|----------|------|---------|
| **Retire** | Turn off what you don't need | Decommission old apps |
| **Retain** | Keep on-premise (no migration) | Legacy systems not worth moving |
| **Relocate** | Move as-is to cloud/different region | VMware to AWS |
| **Rehost** (Lift & Shift) | Move to AWS without changes | EC2 direct migration |
| **Replatform** (Lift & Reshape) | Move + use managed services | DB → RDS |
| **Repurchase** (Drop & Shop) | Switch to SaaS | CRM → Salesforce |
| **Refactor** | Re-architect the application | Monolith → microservices |

Migration tools: **Application Discovery Service** (plan), **MGN** (migrate), **Migration Evaluator** (cost analysis), **Migration Hub** (track).

---

## 18. 🏗️ Well-Architected Framework & Ecosystem

### 6 Pillars

```
┌──────────────────────────────────────────────────────────┐
│              Well-Architected Framework                    │
│                                                           │
│  1. Operational Excellence                                │
│     IaC, small frequent changes, anticipate failure       │
│     learn from failure, use managed services               │
│                                                           │
│  2. Security                                              │
│     Least privilege, no long-term creds, traceability     │
│     encrypt everything, automate, keep people from data    │
│                                                           │
│  3. Reliability                                           │
│     Test recovery, auto-heal, scale horizontally          │
│     stop guessing capacity, automate changes               │
│                                                           │
│  4. Performance Efficiency                                │
│     Go global in minutes, use serverless, experiment      │
│     stay aware of new services                             │
│                                                           │
│  5. Cost Optimization                                     │
│     Pay for what you use, measure efficiency              │
│     use managed services, analyze costs                    │
│                                                           │
│  6. Sustainability                                        │
│     Maximize utilization, adopt efficient tech            │
│     reduce downstream impact                               │
└──────────────────────────────────────────────────────────┘
```

- **Well-Architected Tool**: review your architecture against the 6 pillars → get reports/advice
- **Customer Carbon Footprint Tool**: track/measure your emissions

### CAF = Cloud Adoption Framework

**6 Perspectives** (must remember):

| Perspective | Focus |
|------------|-------|
| **Business** | Ensure cloud investment accelerates business outcomes |
| **People** | Bridge between tech and business |
| **Governance** | Orchestrate initiatives, minimize risks |
| **Platform** | Build scalable, modern cloud platform |
| **Security** | Confidentiality, integrity, availability |
| **Operations** | Deliver cloud services at expected level |

**Transformation Phases**: Envision → Align → Launch → Scale

### Right Sizing

Match resources to actual needs — bigger is NOT always better. Scale up/down is easy in cloud.

### AWS Ecosystem

- Free resources: docs, blog, whitepapers, Solutions Library
- **APN** (Partner Network): Technology + Consulting + Training partners
- **IQ**: find professionals for AWS projects
- **re:Post**: community forum (not for urgent issues)
- **Knowledge Center**: FAQ, troubleshooting, best practices
- **AMS** (Managed Services): AWS experts manage your infra 24/365

---

## 📊 Stats

```yaml
Total time: ~20h (exam prep)
Used in: CLF-C02 study; [[concepts/aws/aws-fundamentals]]
```

---

**Last update**: 2026-03-13
