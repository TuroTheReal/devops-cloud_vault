# Project Workflow: Claude + Obsidian + Git

## 🎯 Overview

```
Design (Claude.ai) → Implementation (Claude Code) → Docs (Obsidian) → Git
```

---

## 📋 PHASE 1: Preparation (15 min)

### Key questions
```
- Goal? [Lab / Portfolio / Production]
- Stack? [AWS/Azure, K8s/ECS, Terraform, etc.]
- Constraints? [Budget, timeline, complexity]
```

### Local structure
```bash
mkdir -p ~/projects/[project-name]/{infra,k8s,apps,docs,scripts}
cd ~/projects/[project-name]
git init
```

---

## 📋 PHASE 2: Design (Claude.ai, 30-60 min)

### 1. Create Claude Project
```
Project → Create
Name: "Infrastructure - [Project]"
Custom Instructions: See template below
```

### 2. Custom Instructions
```markdown
CONTEXT
Project: [Name]
Type: [Lab/Portfolio/Prod]
Stack: [Technologies]
Constraints: Budget [X€/month], Timeline [X weeks]

MODE
Assume DevOps/Cloud basics (post-42).
Focus: production-ready, security, costs.

REVIEW
- Security (secrets, SG, permissions)
- Performance (resources, caching)
- Costs (monthly estimate)
- Maintainability (DRY, docs)
```

### 3. Upload Knowledge Base
```
To upload:
✅ docs/architecture.md (empty template OK)
✅ docs/conventions.md (your standards)
❌ No code at start
```

### 4. Design Session
```markdown
Prompt:

"Project: [NAME]

GOAL
[Clear description]

CONSTRAINTS
- Budget: [X€/month]
- Timeline: [X weeks]

STACK
- Cloud: AWS
- Compute: [ECS/EKS/EC2]
- Database: [RDS/DynamoDB]

NEED
1. Propose architecture with alternatives
2. Estimate monthly costs per option
3. Identify risks
4. Recommend MVP

Don't generate code, let's validate the approach first."
```

### 5. Document decisions (Obsidian)
```markdown
# projects/[project]/decisions.md

## ADR-001: Compute choice (2025-12-23)

**Options**
1. ECS Fargate: ~100€/month, simple
2. EKS: ~150€/month, flexible
3. EC2: ~50€/month, complex

**Decision**: ECS Fargate

**Rationale**
- Lab/Portfolio, focus app not infra
- Budget OK
- Quick start

**Trade-offs**
+ Simplicity
- AWS lock-in
```

---

## 📋 PHASE 3: Implementation (Claude Code, 1-3h)

### 1. Create architecture.md (Obsidian)
```markdown
# [Project] - Architecture

## ASCII Diagram
[Simple diagram]

## Components
- Frontend: S3 + CloudFront (~5€/month)
- Backend: ECS Fargate (~80€/month)
- Database: RDS (~25€/month)
- Total: ~110€/month

## Security Groups
[Main rules]

## Phases
- [ ] Phase 1: Infrastructure (Week 1)
- [ ] Phase 2: Application (Week 2)
- [ ] Phase 3: CI/CD (Week 3)
```

### 2. Implement (Claude Code)
```bash
cd ~/projects/[project]/infra/terraform/

claude "Init Terraform for architecture in docs/architecture.md.

Structure:
- main.tf, network.tf, compute.tf, database.tf
- Remote state S3 + DynamoDB
- Variables, outputs
- Standardized tags

Production-ready, commented FR."
```

### 3. Review generated files
```bash
# Review each file
cat network.tf

# Test syntax
terraform init
terraform validate
terraform plan

# Corrections if needed with Claude Code
claude "Add second NAT Gateway for HA"
```

### 4. Review with Claude.ai
```markdown
"Review this Terraform:

[Paste main files]

Checklist:
- Security (SG, encryption, secrets)
- Terraform best practices
- Optimized costs
- Naming conventions

Point out good AND bad."
```

### 5. Commit
```bash
git add infra/
git commit -m "feat(infra): Add Terraform AWS infra

- VPC with subnets
- ECS Fargate cluster
- RDS PostgreSQL
- Estimated: ~110€/month"
git push
```

---

## 📋 PHASE 4: Documentation (Obsidian, 45-60 min with simplified templates)

### projects/[project]/README.md
```markdown
# [Project]

## Status
🟢 Phase 1 done
🟡 Phase 2 in progress

## Quick Start
```bash
cd infra/terraform
terraform apply
```

## Monitoring
- Dashboard: [URL]
- App: [URL]

## Troubleshooting
See [[troubleshooting/[project]-issues]]
```

### projects/[project]/conventions.md
```markdown
# Conventions

## Naming
Format: [project]-[env]-[resource]-[name]
Ex: myapp-prod-vpc-main

## Git Commits
feat(scope): description
fix(scope): description

## Terraform
- Required variables
- Standard tags
- FR comments for logic
```

### Upload to Claude KB
```
Re-upload to Project:
- architecture.md (updated)
- conventions.md
```

---

## 📋 PHASE 5: Daily Workflow

### For each feature

```
1. DESIGN (Claude.ai, 15 min)
   "Feature [X], propose approach"

2. CODE (Claude Code, 1-2h)
   Implement + local tests

3. REVIEW (Claude.ai, 15 min)
   Review code, security checklist

4. DOC (Obsidian, 15 min)
   Update architecture.md
   Document learnings

5. COMMIT (Git, 5 min)
   Clear message, push
```

### If blocked >30 min
```
1. Create troubleshooting/[date]-[issue].md
2. Ask Claude (detailed context)
3. Apply solution
4. Update troubleshooting note
```

---

## 📋 PHASE 6: Closure (Retrospective)

### projects/[project]/retrospective.md
```markdown
# Retrospective

Date: 2026-01-23
Duration: 4 weeks
Hours: ~80h

## Goals vs Achieved
- [x] App deployed AWS
- [x] Automated CI/CD
- [ ] Multi-region (deprioritized)

## What worked well
✅ Claude + Obsidian workflow
✅ ECS Fargate = simple
✅ Documentation as we go

## What went wrong
❌ Underestimated networking (2 days lost)
❌ Budget exceeded: 130€ vs 110€ planned

## Technical learnings
- [[aws-vpc-advanced]] truly mastered
- [[terraform-modules]] reusable

## Concepts to deepen
- [ ] [[eks-basics]] for next project
- [ ] [[aws-cost-optimization]]

## Reusable
- Terraform modules → ~/terraform-modules/
- Scripts → ~/scripts-library/
- CI template → ~/ci-templates/

## Next Steps
- [ ] Multi-region setup
- [ ] Auto-scaling
- [ ] CDN for assets
```

---

## ✅ Complete Checklist

### Setup
- [ ] Define scope & constraints
- [ ] Filesystem structure + Git
- [ ] Claude.ai Project created
- [ ] Custom Instructions configured
- [ ] Initial KB uploaded

### Design
- [ ] Design session with Claude
- [ ] Documented decisions (ADR)
- [ ] architecture.md created
- [ ] Costs validated

### Implementation
- [ ] Code with Claude Code
- [ ] Local tests OK
- [ ] Claude.ai review
- [ ] Commit with clear message

### Documentation (simplified templates = 65% time saved)
- [ ] Project learnings.md filled (45-60 min with project-template)
- [ ] Concepts extracted (30-40 min/concept with concept-template)
- [ ] Optional cheatsheet updated (20-30 min with cheatsheet-template)
- [ ] architecture.md synchronized
- [ ] Claude KB up to date

### Closure
- [ ] Reusable code extracted
- [ ] Portfolio updated
- [ ] Project archived

---

**Average time simple project: 40-60h over 2-3 weeks**
**Average time complex project: 80-120h over 4-6 weeks**

**Post-project documentation**:
- **Before**: 7h30 (complex templates)
- **After**: 1h30 (simplified templates)
- **Gain**: 80% time saved ✅

**See**: [[../guides/templates-usage-guide]] for quick reference
