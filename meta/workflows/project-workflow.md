# Workflow Projet : Claude + Obsidian + Git

## 🎯 Vue d'ensemble

```
Design (Claude.ai) → Implémentation (Claude Code) → Doc (Obsidian) → Git
```

---

## 📋 PHASE 1 : Préparation (15 min)

### Questions clés
```
- Objectif ? [Lab / Portfolio / Production]
- Stack ? [AWS/Azure, K8s/ECS, Terraform, etc.]
- Contraintes ? [Budget, timeline, complexité]
```

### Structure locale
```bash
mkdir -p ~/projects/[project-name]/{infra,k8s,apps,docs,scripts}
cd ~/projects/[project-name]
git init
```

---

## 📋 PHASE 2 : Design (Claude.ai, 30-60 min)

### 1. Créer Project Claude
```
Project → Create
Nom : "Infrastructure - [Project]"
Custom Instructions : Voir template ci-dessous
```

### 2. Custom Instructions
```markdown
CONTEXTE
Projet : [Nom]
Type : [Lab/Portfolio/Prod]
Stack : [Technologies]
Contraintes : Budget [X€/mois], Timeline [X semaines]

MODE
Assume bases DevOps/Cloud (post-42).
Focus : production-ready, sécurité, coûts.

REVIEW
- Sécurité (secrets, SG, permissions)
- Performance (ressources, caching)
- Coûts (estimation mensuelle)
- Maintenabilité (DRY, docs)
```

### 3. Upload Knowledge Base
```
À uploader :
✅ docs/architecture.md (template vide OK)
✅ docs/conventions.md (vos standards)
❌ Pas de code au début
```

### 4. Session Design
```markdown
Prompt :

"Projet : [NOM]

OBJECTIF
[Description claire]

CONTRAINTES
- Budget : [X€/mois]
- Timeline : [X semaines]

STACK
- Cloud : AWS
- Compute : [ECS/EKS/EC2]
- Database : [RDS/DynamoDB]

BESOIN
1. Propose architecture avec alternatives
2. Estime coûts mensuels par option
3. Identifie risques
4. Recommande MVP

Ne génère PAS de code, validons l'approche d'abord."
```

### 5. Documenter décisions (Obsidian)
```markdown
# projects/[project]/decisions.md

## ADR-001: Choix compute (2025-12-23)

**Options**
1. ECS Fargate : ~100€/mois, simple
2. EKS : ~150€/mois, flexible
3. EC2 : ~50€/mois, complexe

**Décision** : ECS Fargate

**Rationale**
- Lab/Portfolio, focus app pas infra
- Budget OK
- Démarrage rapide

**Trade-offs**
+ Simplicité
- Lock-in AWS
```

---

## 📋 PHASE 3 : Implémentation (Claude Code, 1-3h)

### 1. Créer architecture.md (Obsidian)
```markdown
# [Project] - Architecture

## Diagram ASCII
[Diagramme simple]

## Composants
- Frontend : S3 + CloudFront (~5€/mois)
- Backend : ECS Fargate (~80€/mois)
- Database : RDS (~25€/mois)
- Total : ~110€/mois

## Security Groups
[Rules principales]

## Phases
- [ ] Phase 1 : Infrastructure (Semaine 1)
- [ ] Phase 2 : Application (Semaine 2)
- [ ] Phase 3 : CI/CD (Semaine 3)
```

### 2. Implémenter (Claude Code)
```bash
cd ~/projects/[project]/infra/terraform/

claude "Init Terraform pour architecture dans docs/architecture.md.

Structure :
- main.tf, network.tf, compute.tf, database.tf
- Remote state S3 + DynamoDB
- Variables, outputs
- Tags standardisés

Production-ready, commenté FR."
```

### 3. Review files générés
```bash
# Review chaque fichier
cat network.tf

# Test syntax
terraform init
terraform validate
terraform plan

# Corrections si besoin avec Claude Code
claude "Ajoute second NAT Gateway pour HA"
```

### 4. Review avec Claude.ai
```markdown
"Review ce Terraform :

[Coller fichiers principaux]

Checklist :
- Sécurité (SG, encryption, secrets)
- Best practices Terraform
- Coûts optimisés
- Naming conventions

Pointe bien ET mal."
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

## 📋 PHASE 4 : Documentation (Obsidian, 30 min)

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
- Dashboard : [URL]
- App : [URL]

## Troubleshooting
See [[troubleshooting/[project]-issues]]
```

### projects/[project]/conventions.md
```markdown
# Conventions

## Naming
Format : [project]-[env]-[resource]-[name]
Ex: myapp-prod-vpc-main

## Git Commits
feat(scope): description
fix(scope): description

## Terraform
- Variables obligatoires
- Tags standards
- Commentaires FR pour logique
```

### Upload dans Claude KB
```
Re-upload dans Project :
- architecture.md (updated)
- conventions.md
```

---

## 📋 PHASE 5 : Workflow Quotidien

### Pour chaque feature

```
1. DESIGN (Claude.ai, 15 min)
   "Feature [X], propose approche"

2. CODE (Claude Code, 1-2h)
   Implémenter + tests locaux

3. REVIEW (Claude.ai, 15 min)
   Review code, checklist sécurité

4. DOC (Obsidian, 15 min)
   Update architecture.md
   Learnings documentés

5. COMMIT (Git, 5 min)
   Message clair, push
```

### Si bloqué >30 min
```
1. Créer troubleshooting/[date]-[issue].md
2. Demander Claude (contexte détaillé)
3. Appliquer solution
4. Update troubleshooting note
```

---

## 📋 PHASE 6 : Clôture (Retrospective)

### projects/[project]/retrospective.md
```markdown
# Retrospective

Date : 2026-01-23
Durée : 4 semaines
Heures : ~80h

## Objectifs vs Réalisé
- [x] App déployée AWS
- [x] CI/CD automatisé
- [ ] Multi-région (dépriorisé)

## Ce qui a bien marché
✅ Workflow Claude + Obsidian
✅ ECS Fargate = simple
✅ Documentation au fil de l'eau

## Ce qui a mal tourné
❌ Underestimation networking (2 jours perdus)
❌ Budget dépassé : 130€ vs 110€ prévu

## Learnings techniques
- [[aws-vpc-advanced]] vraiment maîtrisé
- [[terraform-modules]] réutilisables

## Concepts à approfondir
- [ ] [[eks-basics]] pour prochain projet
- [ ] [[aws-cost-optimization]]

## Réutilisable
- Modules Terraform → ~/terraform-modules/
- Scripts → ~/scripts-library/
- CI template → ~/ci-templates/

## Next Steps
- [ ] Multi-région setup
- [ ] Auto-scaling
- [ ] CDN pour assets
```

---

## ✅ Checklist Complète

### Setup
- [ ] Définir scope & contraintes
- [ ] Structure filesystem + Git
- [ ] Project Claude.ai créé
- [ ] Custom Instructions configurées
- [ ] KB initiale uploadée

### Design
- [ ] Session design avec Claude
- [ ] Décisions documentées (ADR)
- [ ] Architecture.md créée
- [ ] Coûts validés

### Implémentation
- [ ] Code avec Claude Code
- [ ] Tests locaux OK
- [ ] Review Claude.ai
- [ ] Commit avec message clair

### Documentation
- [ ] Notes Obsidian à jour
- [ ] Troubleshooting docs
- [ ] Architecture.md synchronized
- [ ] KB Claude à jour

### Clôture
- [ ] Retrospective complète
- [ ] Code réutilisable extrait
- [ ] Portfolio mis à jour
- [ ] Projet archivé

---

**Temps moyen projet simple : 40-60h sur 2-3 semaines**
**Temps moyen projet complexe : 80-120h sur 4-6 semaines**