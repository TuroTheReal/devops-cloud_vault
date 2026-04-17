# DevOps/Cloud Roadmap: École 42 → Cloud Architect

## 🎯 Vision

**Starting point**: 42 student, common core validated
**Destination**: Cloud Architect Freelance (18-24 months)
**Motivation**: Remote, independence

---

## 📅 Timeline & Goals

| Phase | Duration | Goal |
|-------|-------|----------|
| **Phase 1** | Months 1-3 (300h) | Solid fundamentals |
| **Phase 2** | Months 4-6 (300h) | DevOps Intermediate |
| **Phase 3** | Months 7-9 (250h) | SRE Ready |
| **Phase 4** | Months 10-12 (200h) | Cloud Engineer + AWS SA Associate |
| **Phase 5** | Months 13-18 (300h) | Cloud Architect + Viable freelance |

**Total: ~1350h over 18 months** (≈20h/week)

---

## 📚 PHASE 1: Fundamentals (Months 1-3)

### Linux & Shell (60h)
```
✅ Key skills
- Advanced Bash scripting (loops, functions, error handling)
- Networking (TCP/IP, DNS, firewall)
- Systemd, cron, logs

🎯 Validation
Automated backup script with rotation, logs, alerts
```

### Git & CI/CD (40h)
```
✅ Key skills
- Advanced Git (rebase, cherry-pick, submodules)
- GitLab CI: stages, artifacts, cache, environments

🎯 Validation
Working pipeline: build → test → deploy
```

### Docker (80h)
```
✅ Key skills
- Optimized multi-stage Dockerfile
- Docker Compose for complete stack
- Security (scan, rootless, secrets)

🎯 Validation
Containerized microservices app (3+ services)
```

### AWS Basics (80h)
```
✅ Essential services
EC2, VPC, S3, IAM, RDS, ALB, CloudWatch

✅ Key concepts
- Networking (subnets, SG, routing)
- High Availability (Multi-AZ)
- Security (IAM, encryption)

🎯 Validation
3-tier app deployed: ALB + EC2 + RDS
```

### Python DevOps (40h)
```
✅ Key skills
- Scripting (subprocess, argparse)
- AWS SDK (boto3)
- REST APIs

🎯 Validation
AWS automation script (EC2 + S3 + IAM)
```

---

## 🚀 PHASE 2: DevOps Intermediate (Months 4-6)

### Kubernetes (120h) ⭐
```
✅ Core
Pods, Deployments, Services, ConfigMaps, Secrets, Ingress

✅ Advanced
StatefulSets, DaemonSets, RBAC, Helm

✅ Labs
- Complete app on K8s (3+ microservices)
- Prometheus/Grafana monitoring
- Ingress + TLS

🎯 Validation
CKA certification (optional, strongly recommended)
```

### Terraform (100h) ⭐
```
✅ Skills
- Providers, Resources, Modules
- Remote State (S3 + DynamoDB)
- Workspaces (dev/staging/prod)

✅ Lab
Complete AWS infrastructure in Terraform
(VPC, EKS, RDS, monitoring)

🎯 Validation
Production-ready reusable module
```

### Ansible (50h)
```
✅ Skills
- Playbooks, Roles, Handlers
- Ansible Vault (secrets)
- Idempotence

🎯 Validation
Automated server configuration
```

### Advanced CI/CD (30h)
```
✅ Skills
- GitOps (ArgoCD)
- Blue/Green, Canary deployments
- Environments (dev/staging/prod)

🎯 Validation
Complete pipeline to K8s via ArgoCD
```

---

## 📊 PHASE 3: Observability & SRE (Months 7-9)

### Monitoring (80h)
```
✅ Stack
Prometheus, Grafana, Alertmanager

✅ Concepts
- Metrics, PromQL
- SLI, SLO, SLA
- Error budgets

🎯 Validation
Complete stack for K8s app
```

### Logging (60h)
```
✅ Stack
Loki, Promtail, Grafana

✅ Alternative
ELK (concepts)

🎯 Validation
Centralized logs K8s cluster
```

### SRE Practices (70h)
```
✅ Skills
- Incident management
- Runbooks, Post-mortems
- Chaos Engineering
- Capacity planning

🎯 Validation
Complete post-mortem simulated incident
```

### Tracing (40h)
```
✅ Stack
Jaeger, OpenTelemetry

🎯 Validation
Microservices application tracing
```

---

## ☁️ PHASE 4: Multi-Environment Cloud (Months 10-12)

### Advanced AWS (100h) ⭐
```
✅ Services
EKS, ECS/Fargate, ElastiCache, Lambda, CloudFormation

✅ Concepts
- Well-Architected Framework
- Cost Optimization
- Security (KMS, Secrets Manager, GuardDuty)

🎯 Validation
AWS Solutions Architect Associate (mandatory certification)
```

### Multi-Cloud (50h)
```
✅ Basics
Azure (AKS, VMs), GCP (GKE, GCE)

🎯 Validation
Same app deployed AWS + Azure
```

### Architecture Patterns (50h)
```
✅ Patterns
- Microservices patterns
- 12-Factor App
- Event-driven architecture
- Service Mesh (concepts)

🎯 Validation
Complete architecture documentation with diagrams
```

---

## 🏗️ PHASE 5: Cloud Architect (Months 13-18)

### Advanced Architecture (120h)
```
✅ Skills
- Solution Design
- FinOps / Cost Optimization
- Security Architecture (Compliance, Audit)
- Migration strategies

🎯 Validation
AWS Solutions Architect Professional (certification)
3+ documented complex architectures
```

### DevSecOps (80h)
```
✅ Skills
- Security scanning (Trivy, Snyk)
- SAST/DAST
- Secrets management (Vault)
- Compliance (CIS, PCI-DSS)

🎯 Validation
Pipeline with integrated security gates
```

### Soft Skills & Business (100h)
```
✅ Skills
- Technical communication
- Architecture Decision Records
- Estimation (time, cost)
- Freelance basics

✅ Output
- Technical blog (5+ articles)
- Professional GitHub portfolio
- Optimized LinkedIn

🎯 Validation
First freelance contract or Senior CDI
```

---

## 🎓 Recommended Certifications

| Certification | When | Price | Importance |
|---------------|-------|------|------------|
| **CKA** (Kubernetes) | Month 6 | ~$395 | ⭐⭐⭐ Strongly recommended |
| **AWS SA Associate** | Month 12 | ~$150 | ⭐⭐⭐ Mandatory |
| **Terraform Associate** | Months 6-12 | Free | ⭐⭐ Nice to have |
| **AWS SA Professional** | Month 18 | ~$300 | ⭐⭐⭐ For freelance |

**Total certifications: ~$850** (excellent ROI)

---

## 📋 Key Milestones

### ✅ Month 3: Fundamentals
- Containerized app on AWS
- Working CI/CD pipeline
- 3+ GitHub projects

### ✅ Month 6: Intermediate
- Prod-ready app on Kubernetes
- Terraform infrastructure
- Operational monitoring
- (Optional CKA)

### ✅ Month 9: SRE Ready
- Complete observability stack
- Runbooks + post-mortems
- Production-ready application

### ✅ Month 12: Cloud Engineer
- Multi-environments (dev/staging/prod)
- **AWS SA Associate** ✅
- Portfolio 5+ projects
- First DevOps interviews

### ✅ Month 18: Cloud Architect
- 3+ complex architectures
- **AWS SA Professional** ✅
- Active blog
- **Viable freelance**

---

## 🎯 Immediate Action Plan

### This week
- [ ] Setup Ubuntu VM or WSL2
- [ ] AWS Free Tier account
- [ ] Install: Docker, kubectl, terraform
- [ ] Start Linux basics

### This month
- [ ] Linux & Shell (60h)
- [ ] Docker basics (40h/80h)
- [ ] First complex Bash script on GitHub

### In 3 months
- [ ] Phase 1 completed
- [ ] App deployed AWS
- [ ] 10+ Obsidian notes
- [ ] 3 public GitHub projects

---

## 📚 Essential Resources

### Books (free/paid)
- **The Phoenix Project** (DevOps mindset)
- **Site Reliability Engineering** (Google, free online)
- **Kubernetes Up & Running**
- **Terraform: Up & Running**

### Platforms
- **AWS Skill Builder** (free)
- **KodeKloud** (K8s, Terraform - ~$15/month)
- **A Cloud Guru** (certifications - ~$30/month)

### Total learning cost
- Platforms: ~$500 (12 months)
- Certifications: ~$850
- AWS labs: ~$100
**Total: ~$1500** over 18 months

---

## 📊 Tracking Progress

### Weekly metrics
```markdown
Week of [date]:
- Hours: __/20h
- Concepts mastered: __
- Labs completed: __
- GitHub commits: __
```

### Monthly review
```markdown
Month [X]:
- ✅ Goals achieved: __/__
- ⚠️ Difficulties: __
- 📝 Adjustments: __
```

---

**Current status**: Phase 0 (Setup)
**Next action**: Environment setup + start Linux
**Last update**: 2025-12-23
