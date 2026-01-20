# DEVOPS CLOUD VAULT

<p align="center">
  <img src="https://img.shields.io/badge/DevOps-Learning-blue.svg"/>
  <img src="https://img.shields.io/badge/Cloud-AWS%20%7C%20Azure%20%7C%20GCP-orange.svg"/>
  <img src="https://img.shields.io/badge/Status-Active-success.svg"/>
  <img src="https://img.shields.io/badge/License-CC%20BY%204.0-yellow.svg"/>
</p>

<p align="center">
  <i>A comprehensive knowledge vault for DevOps and Cloud technologies - from fundamentals to production-ready skills</i>
</p>

<p align="center">
  aliases: [devops-cloud-vault]
</p>

---

## Table of Contents
- [About](#about)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Learning Paths](#learning-paths)
- [Technologies Covered](#technologies-covered)
- [How to Use This Vault](#how-to-use-this-vault)
- [Learning Philosophy](#learning-philosophy)
- [Progress Tracking](#progress-tracking)
- [Contributing](#contributing)
- [Roadmap](#roadmap)
- [Contact](#contact)

## About

**DevOps Cloud Vault** is a personal knowledge base for mastering DevOps and Cloud technologies. This repository serves as a structured learning environment combining theory, practice, and real-world problem-solving.

> **Note**: This is a personal learning vault - some notes are polished, some are drafts. Feel free to use it as inspiration for your own knowledge base. Projects referenced are personal and may not be public.

### Why This Vault?

In the rapidly evolving DevOps landscape, having organized, searchable, and actionable knowledge is crucial. This vault provides:

- **📚 Structured Learning**: Organized by concepts, tools, and difficulty levels
- **⚡ Quick Reference**: Cheatsheets for instant command lookup
- **🛠️ Hands-On Practice**: Real projects to build portfolio-worthy work
- **🔧 Problem Solving**: Troubleshooting guides for common issues
- **📈 Progress Tracking**: Document learning journey and retention
- **🎯 Career Ready**: Skills aligned with industry requirements

**Target Audience:**
- DevOps engineers building foundational knowledge
- System administrators transitioning to cloud
- Developers adopting DevOps practices
- Students preparing for cloud certifications
- Teams building shared knowledge bases

## Repository Structure

```
devops-cloud_vault/
│
├── 📖 cheatsheets/           # Quick reference cheatsheets/guides
│   ├── docker/               # Docker, Compose, Swarm
│   ├── linux/                # Linux basics & security
│   ├── github/               # Git & GitHub
│   ├── traefik/              # Reverse proxy
│   └── taskfile/             # Task automation
│
├── 💡 concepts/              # Deep dives and theory
│   ├── docker/               # 11 concepts (containers, networks, swarm, backup)
│   ├── networking/           # Fundamentals, OSI Model, ICMP
│   ├── linux/                # Security, firewall (5 concepts)
│   ├── git/                  # Version control fundamentals
│   ├── traefik/              # Reverse proxy concepts
│   └── monitoring/           # Observability basics (2 concepts)
│
├── 🚀 projects/              # Real-world learning reports
│   ├── 2024-11-ft-ping-traceroute/      # Network programming (C, ICMP)
│   ├── 2024-XX-transcendence-monitoring/# Observability (ELK, Prometheus, Grafana)
│   ├── 2025-12-glasck-deployment/       # Docker Swarm production deployment
│   └── 2025-12-vps-hetzner-init-setup/  # VPS setup & hardening
│
├── 🗺️ MOCs/                   # Maps of Content (learning paths)
│   ├── MOC-Docker-Production.md         # Container orchestration
│   ├── MOC-Linux-Security.md            # System hardening
│   └── MOC-Networking-Fundamentals.md   # OSI, ICMP, Docker networks
│
├── 🔧 troubleshooting/       # Debug guides (planned)
│
└── 📋 meta/                  # Templates and guides
    ├── templates/            # Document templates
    ├── guides/               # Usage guides
    └── workflows/            # Learning workflows
```

### Section Overview

| Section | Purpose | Use When |
|---------|---------|----------|
| [**MOCs**](MOCs/) | Structured learning paths | Starting a new topic, need roadmap |
| [**Concepts**](concepts/) | Deep understanding | Learning how things work internally |
| [**Cheatsheets**](cheatsheets/) | Quick command reference | Need syntax fast, daily operations |
| [**Projects**](projects/) | Real-world learnings | Want to see practical applications |
| [**Meta**](meta/) | Templates & guides | Creating new content |

## Getting Started

### Choose Your Path

```bash
# New to DevOps / Starting with containers?
Start with: MOCs/MOC-Docker-Production.md
Then explore: concepts/docker/ + cheatsheets/docker/

# Want to understand networking?
Start with: MOCs/MOC-Networking-Fundamentals.md
Then explore: concepts/networking/ + projects/ft-ping-traceroute/

# Setting up a VPS / Linux hardening?
Start with: MOCs/MOC-Linux-Security.md
Then explore: concepts/linux/ + projects/vps-hetzner-init/

# Learning from real projects?
Browse: projects/ folder for real-world learning reports
```

### Clone and Explore

```bash
# Clone this repository
git clone https://github.com/TuroTheReal/devops-cloud_vault.git
cd devops-cloud_vault

# Explore structure
ls -la
tree -L 2  # if tree is installed

# Start with a cheatsheet
cat cheatsheets/docker/docker-commands.md

# Read a concept
cat concepts/docker/docker-why-containers.md  # (coming soon)
```

## Learning Paths

### 🐳 Path 1: Docker Production

**Goal**: Master containerization from basics to production

**Completed learnings**:
- Docker fundamentals (containers, images, layers, volumes)
- Multi-container orchestration with Docker Compose
- Production clustering
- Overlay networks, service discovery, rolling updates
- Traefik reverse proxy with automatic SSL

**Time Invested**: ~68h
**Projects**: [Glasck Deployment](projects/2025-12-glasck-deployment/), [Docker concepts](concepts/docker/)
**MOC**: [MOC-Docker-Production](MOCs/MOC-Docker-Production.md)

---

### 🌐 Path 2: Networking Fundamentals

**Goal**: Understand network protocols from Layer 3 to application layer

**Completed learnings**:
- OSI Model (Layer 3, 4, 7)
- ICMP protocol implementation (ping, traceroute)
- TCP/IP, DNS, ports, firewalls
- Docker networking (bridge, overlay, service mesh)

**Time Invested**: ~90h
**Projects**: [ft_ping & ft_traceroute](projects/2024-11-ft-ping-traceroute/)
**MOC**: [MOC-Networking-Fundamentals](MOCs/MOC-Networking-Fundamentals.md)

---

### 🔐 Path 3: Linux Security

**Goal**: Secure and harden Linux VPS for production use

**Completed learnings**:
- SSH hardening (key-based auth, fail2ban)
- UFW firewall configuration
- User management and sudo hardening
- System monitoring and log analysis
- VPS initial setup (Hetzner)

**Time Invested**: ~15h
**Projects**: [VPS Hetzner Init Setup](projects/2025-12-vps-hetzner-init-setup/)
**MOC**: [MOC-Linux-Security](MOCs/MOC-Linux-Security.md)

---

### 📋 Planned Paths (Not Yet Started)

**☸️ Kubernetes** (planned):
- Pod networking, Services, Ingress
- StatefulSets, ConfigMaps, Secrets
- Helm charts, operators

**☁️ Cloud Platforms** (AWS focus):
- VPC, EC2, S3, RDS
- ECS, EKS (container orchestration)
- IAM, CloudFormation

**🏗️ Infrastructure as Code**:
- Terraform (state management, modules)
- Ansible (playbooks, roles)

**🔄 CI/CD & GitOps**:
- GitHub Actions
- ArgoCD for GitOps

## Technologies Covered

### 🐳 Containerization & Orchestration
- **Docker**: Container lifecycle, images, layers, volumes, networks
- **Docker Compose**: Multi-container apps, healthchecks
- **Docker Swarm**: Discovering : clustering, overlay networks, rolling updates, secrets
- **Best Practices**: Security, optimization, multi-stage builds

### 🌐 Networking
- **Fundamentals**: TCP/IP, DNS, ports, firewalls
- **OSI Model**: Layer 3 (ICMP), Layer 4 (TCP/UDP), Layer 7 (HTTP)
- **Network Programming**: Raw sockets, ICMP protocol (ping, traceroute)
- **Docker Networking**: Bridge, overlay, service discovery, routing mesh

### 🔐 Linux & Security
- **System Administration**: UFW firewall, SSH hardening, Fail2ban
- **User Management**: Permissions, sudo configuration
- **Monitoring**: System metrics, log analysis
- **VPS Setup**: Hetzner deployment, security baseline

### 🔄 Version Control & Collaboration
- **Git Fundamentals**: Local vs remote, branching, merging, rebasing
- **GitHub Workflows**: Feature branches, pull requests, stash management
- **Best Practices**: Force push safety, commit conventions, conflict resolution
- **Advanced Git**: Cherry-pick, reflog, detached HEAD recovery

### 🔄 Reverse Proxy & Load Balancing
- **Traefik**: Automatic SSL (Let's Encrypt), service discovery, path-based routing
- **Layer 7 routing**: HTTP load balancing across containers

### 📝 Learning & Documentation ✅
- **MOCs**: Structured learning paths for complex topics
- **Project Learnings**: Real-world experience extraction
- **Templates**: Reusable documentation patterns

### 📋 Planned (Not Yet Started)
- ☁️ **Cloud Platforms**: AWS VPC, EC2, S3 (planned)
- ☸️ **Kubernetes**: Pod networking, Services, Ingress (planned)
- 🏗️ **IaC**: Terraform, Ansible (planned)
- 🔄 **CI/CD**: GitHub Actions, GitOps (planned)
- 📊 **Observability**: Prometheus, Grafana (planned)

## How to Use This Vault

### 1. Quick Command Lookup
```bash
# Need a Docker command fast?
→ cheatsheets/docker/docker.md

# Kubernetes syntax?
→ cheatsheets/kubernetes/kubectl.md
```

### 2. Learning New Technology
```bash
# Start with concept to understand WHY
→ concepts/terraform/infrastructure-as-code.md

# Then quick reference for HOW
→ cheatsheets/terraform/terraform.md

# Finally practice with project
→ projects/intermediate/terraform-aws/
```

### 3. Debugging Issues
```bash
# Search for error in troubleshooting
→ troubleshooting/kubernetes/pod-failures.md

# Understand root cause in concepts
→ concepts/kubernetes/pod-lifecycle.md

# Verify fix with cheatsheet commands
→ cheatsheets/kubernetes/kubectl.md
```

## Learning Philosophy

### 70-30 Rule Target
- **70% Hands-On**: Building, breaking, fixing
- **30% Assisted**: Claude, docs, tutorials

### Active Learning Cycle
```
1. Understand concept → 2. Try it → 3. Break it → 4. Fix it → 5. Document it
         ↑                                                            ↓
         └────────────────────── Repeat with variations ─────────────┘
```

### Retention Strategy
- **Day 0**: Initial learning
- **Day 1**: Quick review
- **Day 3**: Practice variation
- **Day 7**: Retention test
- **Day 30**: Full review

### Documentation Standards
Every document includes:
- Clear objectives
- Prerequisites
- Hands-on examples
- Common pitfalls
- Personal notes and learnings

## Progress Tracking

### Learning Metrics

Track your progress in each document:
- **Time Invested**: Total hours spent
- **Claude Ratio**: Assisted vs independent time
- **Retention Score**: Self-assessment after 7/30 days
- **Projects Built**: Practical applications
- **Errors Fixed**: Troubleshooting experience

### Status Indicators
```
⏳ Learning      - Active study phase
✅ Learned       - Hands-on application
🎓 Mastered      - Can teach and apply
```

### Certification Prep
This vault supports preparation for:
- **AWS Certified Solutions Architect**
- **Certified Kubernetes Administrator (CKA)**
- **Terraform Associate**
- **Docker Certified Associate**

## Coming Next

- [ ] Add troubleshooting guides for Docker/networking issues
- [ ] Create Kubernetes learning path MOC
- [ ] Start AWS VPC networking concepts
- [ ] Add CI/CD concepts (GitHub Actions)
- [ ] Monitoring basics (Prometheus/Grafana concepts)

## Statistics

- **Total Documents**: 52 active documents
- **Cheatsheets**: 8 (Docker × 3, Linux × 2, Git/GitHub, Traefik, Taskfile)
- **Concepts**: 22 (Docker ecosystem, Networking, Linux security, Git/GitHub)
- **Projects**: 4 real-world learning reports (~300h documented)
- **MOCs**: 3 learning paths (Docker Production, Linux Security, Networking)
- **Troubleshooting**: Planned for future
- **Last Updated**: 2026-01-20

## Resources

### Official Documentation
- [Docker Docs](https://docs.docker.com/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Terraform Docs](https://terraform.io/docs/)
- [Ansible Docs](https://docs.ansible.com/)
- [AWS Docs](https://docs.aws.amazon.com/)

### Learning Platforms
- [KillerCoda](https://killercoda.com/) - Interactive scenarios
- [A Cloud Guru](https://acloudguru.com/) - Video courses
- [Linux Academy](https://linuxacademy.com/) - Hands-on labs

### Community
- [r/devops](https://reddit.com/r/devops)
- [DevOps Stack Exchange](https://devops.stackexchange.com/)
- [CNCF Slack](https://cloud-native.slack.com/)

## Contact

- **GitHub**: [@TuroTheReal](https://github.com/TuroTheReal)
- **Email**: arthurbernard.dev@gmail.com
- **LinkedIn**: [Arthur Bernard](https://www.linkedin.com/in/arthurbernard92/)

---

<p align="center">
  <b>⭐ Star this repo if you find it useful!</b>
</p>

<p align="center">
  <i>Built with 🧠 for continuous learning and 💪 for career growth</i>
</p>

<p align="center">
  Made with focus by <a href="https://github.com/TuroTheReal">Arthur Bernard</a>
</p>

---

**License**: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) - Free to use and adapt with attribution
**Version**: 2.2
**Last Updated**: 2026-01-20
