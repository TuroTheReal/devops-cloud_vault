---
tags:
  - meta
  - backlog
  - learning
  - todo
  - planned
created: 2026-05-04
updated: 2026-05-04
status: active
---

# 📝 Learning Backlog — Prochains concepts à écrire

> **Quoi**: Liste consolidée des fiches à créer, extraite des `(planned)` des MOCs et des sections `📝 TODO` des projets.
>
> **Pourquoi**: Vue d'ensemble de ce qui manque pour compléter le vault, sans avoir à fouiller chaque MOC ou projet.
>
> **Comment l'utiliser**: Clique sur un wikilink rouge ci-dessous → Obsidian crée le fichier au bon endroit. Ne supprime pas les liens rouges, ils servent de TODO visuels.

---

## 🌩️ Cloud AWS — 7 concepts à écrire

> Référencés dans [[MOC-Cloud-AWS]] comme `(planned)`.

- [ ] [[concepts/aws/aws-vpc]] — Subnets, route tables, NAT gateways, VPC peering
- [ ] [[concepts/aws/aws-alb]] — Load balancing, routing, target groups & health checks, TLS with ACM
- [ ] [[concepts/aws/aws-s3]] — Buckets, policies, lifecycle rules, static hosting
- [ ] [[concepts/aws/aws-rds]] — RDS setup, backups, Multi-AZ, security
- [ ] [[concepts/aws/aws-secrets-manager]] — Secrets rotation, integration with EC2/Lambda
- [ ] [[concepts/aws/aws-ecs_eks]] — ECS Fargate, task definitions, service discovery
- [ ] [[concepts/aws/aws-lambda]] — Functions, triggers, API Gateway integration

**Phase**: après `aws-fundamentals` ✅ → ces 7 concepts pour atteindre niveau intermédiaire AWS.
**Lié à**: AWS Solutions Architect Associate (cf [[roadmap]] Phase 4)

---

## 🏗️ Infrastructure as Code — 5 concepts à écrire

> Référencés dans [[MOC-Infrastructure-as-Code]] comme `(planned)`.

- [ ] [[concepts/terraform/terraform-modules]] — Réutilisabilité, abstraction, versioning
- [ ] [[concepts/terraform/terraform-remote-state]] — S3 + DynamoDB, locking, équipes
- [ ] [[concepts/terraform/terraform-cicd]] — GitHub Actions, validation, plan/apply automatisés
- [ ] [[concepts/ansible/ansible-vault]] — Secrets chiffrés, gestion mots de passe
- [ ] [[concepts/ansible/ansible-roles]] — Structuration playbooks, réutilisabilité

**Phase**: après `terraform-fundamentals` ✅ + `ansible-fundamentals` ✅.
**Lié à**: Terraform Associate certif (optionnel)

---

## 🔄 CI/CD & Git — 1 concept à écrire

> Référencé dans [[git-github-fundamentals]] et [[git-semantic-versioning]].

- [ ] [[concepts/git/ci-cd-basics]] — GitHub Actions, GitOps, pipeline build → test → deploy

**Lié à**: futur [[MOC-CICD-Pipeline]] (à créer aussi)

---

## 🐳 Docker — 1 concept à écrire

> Référencé dans [[docker-compose-healthchecks]].

- [ ] [[concepts/docker/docker-swarm-healthchecks]] — Healthchecks dans Swarm, différences avec Compose

---

## 🐧 Linux — 1 concept à écrire

> Référencé dans [[linux-basics-commands]].

- [ ] [[concepts/linux/linux-security]] — Synthèse sécurité (regroupe SSH hardening, UFW, Fail2ban en méta-concept)

---

## 🔧 Outils — 1 concept à écrire

> Référencé dans [[taskfile-commands]].

- [ ] [[concepts/tools/makefile]] — Make basics, comparaison avec Taskfile

---

## ☸️ Kubernetes — concept référencé dans guide

> Référencé dans [[claude-guide]] comme exemple.

- [ ] [[concepts/kubernetes/k8s-controllers]] — Deployments, StatefulSets, DaemonSets

**Lié à**: futur [[MOC-Kubernetes-Fundamentals]]

---

## 📂 Extractions de projets

> Sections `📝 TODO` des fiches projets — à extraire au fur et à mesure des relectures.

### From [[2024-11-ft-ping-traceroute]]

- [ ] [[cheatsheets/linux/networking]] — Commandes networking utiles (ip, ss, ping, traceroute, netstat)
- [ ] [[troubleshooting/networking/icmp-not-working]] — Diagnostic ICMP bloqué
- [ ] [[concepts/networking/raw-sockets-programming]] — Programmation raw sockets en C

### From [[2024-XX-transcendence-monitoring]]

- [ ] [[cheatsheets/monitoring/elk-commands]] — Queries ELK utiles
- [ ] [[cheatsheets/monitoring/prometheus-commands]] — Queries Prometheus + API calls

---

## 🗺️ MOCs futurs à créer

> Référencés dans [[MOC-Docker-Production]] et [[claude-guide]].

- [ ] [[MOCs/MOC-Kubernetes-Fundamentals]] — Pods, Services, Ingress, StatefulSets
- [ ] [[MOCs/MOC-CICD-Pipeline]] — GitHub Actions, GitOps avec ArgoCD

---

## 📊 Stats

| Catégorie                             | À écrire |
| ------------------------------------- | -------- |
| Concepts AWS                          | 7        |
| Concepts IaC                          | 5        |
| Concepts Docker                       | 1        |
| Concepts Linux                        | 1        |
| Concepts Git/CI-CD                    | 1        |
| Concepts Kubernetes                   | 1        |
| Concepts Tools                        | 1        |
| Cheatsheets (extractions projets)     | 3        |
| Troubleshooting (extractions projets) | 1        |
| MOCs futurs                           | 2        |
| **Total**                             | **23**   |

---

## 🎯 Priorités recommandées

### Court terme (déjà partiellement traité)

1. **AWS** : `aws-vpc`, `aws-s3` — utilisés tout le temps en infra cloud
2. **CI/CD** : `ci-cd-basics` — bloquant pour avancer sur GitOps

### Moyen terme

1. **Terraform avancé** : `terraform-modules`, `terraform-remote-state` — production-ready
2. **Kubernetes** : MOC + concepts core — Phase 2 [[roadmap]]

### Long terme (extractions)

1. **Cheatsheets networking + monitoring** — extractions au moment de relire les projets

---

## 🔗 Liens utiles

- [[roadmap]] — Plan carrière long terme (18 mois)
- [[MOCs/MOCS]] — Index de tous les MOCs
- [[CONCEPTS]] — Index des concepts existants

---

**Conseil** : ce backlog est un **point d'entrée**. Quand tu attaques un nouveau concept, ouvre cette fiche, clique sur le lien rouge, et Obsidian crée le fichier au bon endroit avec le bon nom. Mets à jour le ✅ une fois la fiche écrite.
