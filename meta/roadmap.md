# Roadmap DevOps/Cloud : École 42 → Cloud Architect

## 🎯 Vision

**Départ** : Étudiant 42, tronc commun validé
**Arrivée** : Cloud Architect Freelance (18-24 mois)
**Motivation** : Remote, indépendance, 70-120k€+/an

---

## 📅 Timeline & Objectifs

| Phase | Durée | Objectif | Salaire potentiel |
|-------|-------|----------|-------------------|
| **Phase 1** | Mois 1-3 (300h) | Fondamentaux solides | - |
| **Phase 2** | Mois 4-6 (300h) | DevOps Intermediate | 40-50k€ Junior |
| **Phase 3** | Mois 7-9 (250h) | SRE Ready | 50-65k€ |
| **Phase 4** | Mois 10-12 (200h) | Cloud Engineer | 65-80k€ + AWS SA Associate |
| **Phase 5** | Mois 13-18 (300h) | Cloud Architect | 90-120k€ + Freelance viable |

**Total : ~1350h sur 18 mois** (≈20h/semaine)

---

## 📚 PHASE 1 : Fondamentaux (Mois 1-3)

### Linux & Shell (60h)
```
✅ Compétences clés
- Bash scripting avancé (loops, functions, error handling)
- Networking (TCP/IP, DNS, firewall)
- Systemd, cron, logs

🎯 Validation
Script backup automatisé avec rotation, logs, alertes
```

### Git & CI/CD (40h)
```
✅ Compétences clés
- Git avancé (rebase, cherry-pick, submodules)
- GitLab CI : stages, artifacts, cache, environments

🎯 Validation
Pipeline fonctionnel : build → test → deploy
```

### Docker (80h)
```
✅ Compétences clés
- Dockerfile multi-stage optimisé
- Docker Compose pour stack complète
- Security (scan, rootless, secrets)

🎯 Validation
App microservices (3+ services) containerisée
```

### AWS Basics (80h)
```
✅ Services essentiels
EC2, VPC, S3, IAM, RDS, ALB, CloudWatch

✅ Concepts clés
- Networking (subnets, SG, routing)
- High Availability (Multi-AZ)
- Security (IAM, encryption)

🎯 Validation
App 3-tier déployée : ALB + EC2 + RDS
```

### Python DevOps (40h)
```
✅ Compétences clés
- Scripting (subprocess, argparse)
- AWS SDK (boto3)
- APIs REST

🎯 Validation
Script automation AWS (EC2 + S3 + IAM)
```

---

## 🚀 PHASE 2 : DevOps Intermediate (Mois 4-6)

### Kubernetes (120h) ⭐
```
✅ Core
Pods, Deployments, Services, ConfigMaps, Secrets, Ingress

✅ Advanced
StatefulSets, DaemonSets, RBAC, Helm

✅ Labs
- App complète sur K8s (3+ microservices)
- Monitoring Prometheus/Grafana
- Ingress + TLS

🎯 Validation
CKA certification (optionnel, fortement recommandé)
```

### Terraform (100h) ⭐
```
✅ Compétences
- Providers, Resources, Modules
- Remote State (S3 + DynamoDB)
- Workspaces (dev/staging/prod)

✅ Lab
Infrastructure AWS complète en Terraform
(VPC, EKS, RDS, monitoring)

🎯 Validation
Module réutilisable production-ready
```

### Ansible (50h)
```
✅ Compétences
- Playbooks, Roles, Handlers
- Ansible Vault (secrets)
- Idempotence

🎯 Validation
Configuration serveurs automatisée
```

### CI/CD Avancé (30h)
```
✅ Compétences
- GitOps (ArgoCD)
- Blue/Green, Canary deployments
- Environments (dev/staging/prod)

🎯 Validation
Pipeline complet vers K8s via ArgoCD
```

---

## 📊 PHASE 3 : Observabilité & SRE (Mois 7-9)

### Monitoring (80h)
```
✅ Stack
Prometheus, Grafana, Alertmanager

✅ Concepts
- Metrics, PromQL
- SLI, SLO, SLA
- Error budgets

🎯 Validation
Stack complète pour app K8s
```

### Logging (60h)
```
✅ Stack
Loki, Promtail, Grafana

✅ Alternative
ELK (concepts)

🎯 Validation
Logs centralisés cluster K8s
```

### SRE Practices (70h)
```
✅ Compétences
- Incident management
- Runbooks, Post-mortems
- Chaos Engineering
- Capacity planning

🎯 Validation
Post-mortem complet incident simulé
```

### Tracing (40h)
```
✅ Stack
Jaeger, OpenTelemetry

🎯 Validation
Tracing application microservices
```

---

## ☁️ PHASE 4 : Cloud Multi-Environnement (Mois 10-12)

### AWS Avancé (100h) ⭐
```
✅ Services
EKS, ECS/Fargate, ElastiCache, Lambda, CloudFormation

✅ Concepts
- Well-Architected Framework
- Cost Optimization
- Security (KMS, Secrets Manager, GuardDuty)

🎯 Validation
AWS Solutions Architect Associate (certification obligatoire)
```

### Multi-Cloud (50h)
```
✅ Basics
Azure (AKS, VMs), GCP (GKE, GCE)

🎯 Validation
Même app déployée AWS + Azure
```

### Architecture Patterns (50h)
```
✅ Patterns
- Microservices patterns
- 12-Factor App
- Event-driven architecture
- Service Mesh (concepts)

🎯 Validation
Documentation architecture complète avec diagrammes
```

---

## 🏗️ PHASE 5 : Cloud Architect (Mois 13-18)

### Architecture Avancée (120h)
```
✅ Compétences
- Solution Design
- FinOps / Cost Optimization
- Security Architecture (Compliance, Audit)
- Migration strategies

🎯 Validation
AWS Solutions Architect Professional (certification)
3+ architectures complexes documentées
```

### DevSecOps (80h)
```
✅ Compétences
- Security scanning (Trivy, Snyk)
- SAST/DAST
- Secrets management (Vault)
- Compliance (CIS, PCI-DSS)

🎯 Validation
Pipeline avec security gates intégrés
```

### Soft Skills & Business (100h)
```
✅ Compétences
- Communication technique
- Architecture Decision Records
- Estimation (time, cost)
- Freelance basics

✅ Output
- Blog technique (5+ articles)
- Portfolio GitHub professionnel
- LinkedIn optimisé

🎯 Validation
Premier contrat freelance ou CDI Senior
```

---

## 🎓 Certifications Recommandées

| Certification | Quand | Prix | Importance |
|---------------|-------|------|------------|
| **CKA** (Kubernetes) | Mois 6 | ~$395 | ⭐⭐⭐ Fortement recommandé |
| **AWS SA Associate** | Mois 12 | ~$150 | ⭐⭐⭐ Obligatoire |
| **Terraform Associate** | Mois 6-12 | Gratuit | ⭐⭐ Nice to have |
| **AWS SA Professional** | Mois 18 | ~$300 | ⭐⭐⭐ Pour freelance |

**Total certifications : ~$850** (excellent ROI)

---

## 💰 Projection Salaire France

| Niveau | XP | CDI | Freelance (TJM) |
|--------|-----|-----|-----------------|
| Junior DevOps | 0-2 ans | 38-50k€ | 350-450€ |
| DevOps Engineer | 2-4 ans | 50-70k€ | 450-600€ |
| Senior DevOps | 4-6 ans | 70-90k€ | 600-800€ |
| Cloud Architect | 6+ ans | 90-120k€ | 800-1200€ |

**Remote** : Possible dès Junior
**Freelance viable** : Dès 2 ans d'XP (Mois 12-18 de ce roadmap)

---

## 📋 Milestones Clés

### ✅ Mois 3 : Fondamentaux
- App containerisée sur AWS
- Pipeline CI/CD fonctionnel
- 3+ projets GitHub

### ✅ Mois 6 : Intermediate
- App sur Kubernetes prod-ready
- Infrastructure Terraform
- Monitoring opérationnel
- (CKA optionnel)

### ✅ Mois 9 : SRE Ready
- Stack observabilité complète
- Runbooks + post-mortems
- Application production-ready

### ✅ Mois 12 : Cloud Engineer
- Multi-environnements (dev/staging/prod)
- **AWS SA Associate** ✅
- Portfolio 5+ projets
- Premiers entretiens DevOps

### ✅ Mois 18 : Cloud Architect
- 3+ architectures complexes
- **AWS SA Professional** ✅
- Blog actif
- **Freelance viable**

---

## 🎯 Plan Action Immédiat

### Cette semaine
- [ ] Setup Ubuntu VM ou WSL2
- [ ] Compte AWS Free Tier
- [ ] Install : Docker, kubectl, terraform
- [ ] Démarrer Linux basics

### Ce mois
- [ ] Linux & Shell (60h)
- [ ] Docker basics (40h/80h)
- [ ] Premier script Bash complexe sur GitHub

### Dans 3 mois
- [ ] Phase 1 complétée
- [ ] App déployée AWS
- [ ] 10+ notes Obsidian
- [ ] 3 projets GitHub publics

---

## 📚 Ressources Essentielles

### Livres (gratuits/payants)
- **The Phoenix Project** (DevOps mindset)
- **Site Reliability Engineering** (Google, gratuit online)
- **Kubernetes Up & Running**
- **Terraform: Up & Running**

### Plateformes
- **AWS Skill Builder** (gratuit)
- **KodeKloud** (K8s, Terraform - ~$15/mois)
- **A Cloud Guru** (certifications - ~$30/mois)

### Coût total apprentissage
- Plateformes : ~$500 (12 mois)
- Certifications : ~$850
- AWS labs : ~$100
**Total : ~$1500** sur 18 mois

**ROI** : Premier salaire DevOps = 40k€+

---

## 📊 Tracking Progress

### Metrics hebdomadaires
```markdown
Semaine du [date] :
- Heures : __/20h
- Concepts maîtrisés : __
- Labs complétés : __
- Commits GitHub : __
```

### Review mensuelle
```markdown
Mois [X] :
- ✅ Objectifs atteints : __/__
- ⚠️ Difficultés : __
- 📝 Ajustements : __
```

---

**Status actuel** : Phase 0 (Setup)
**Prochaine action** : Setup environnement + démarrer Linux
**Dernière update** : 2025-12-23