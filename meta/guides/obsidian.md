# Setup Obsidian pour DevOps/Cloud

## 🎯 Objectif

Knowledge base optimale : organisation claire, liens entre concepts, synergie avec Claude

---

## 📁 Structure du Vault (Minimal & Scalable)

```
~/devops-vault/
│
├── 00-meta/                    # Meta & templates
│   ├── templates/
│   │   ├── concept.md
│   │   └── cheatsheet.md
│   └── workflows/
│       └── learning-workflow.md
│
├── concepts/                   # Notes par concept
│   ├── kubernetes/
│   ├── terraform/
│   ├── aws/
│   ├── docker/
│   └── cicd/
│
├── cheatsheets/               # Référence rapide
│   ├── kubectl.md
│   ├── terraform.md
│   └── aws-cli.md
│
├── projects/                  # Documentation projets
│   └── [project-name]/
│       ├── architecture.md
│       ├── decisions.md
│       └── retrospective.md
│
├── troubleshooting/          # Bugs résolus
│   └── YYYY-MM-DD-issue.md
│
├── daily/                    # Notes quotidiennes (optionnel)
│
└── attachments/              # Images, diagrammes
```

---

## 🏷️ Système de Tags (Simple)

### Tags de statut
```yaml
tags: [status/learning]    # En apprentissage
tags: [status/mastered]    # Maîtrisé
tags: [status/review]      # À réviser
```

### Tags de type
```yaml
tags: [concept, kubernetes]
tags: [cheatsheet, docker]
tags: [project, portfolio]
```

### Exemple complet
```yaml
---
tags: [concept, kubernetes, status/learning]
created: 2025-12-23
difficulty: ⭐⭐ (2/5)
time-to-master: 4h
---
```

---

## 🔗 Conventions de Liens

### Lien simple
```markdown
Voir [[k8s-pods]] pour les bases
```

### Lien avec alias
```markdown
Les [[k8s-pods|pods]] sont les unités de base
```

### Règles de nommage
```
Concepts : [techno]-[concept].md
  Ex: k8s-deployments.md, aws-vpc.md

Cheatsheets : [tool]-commands.md
  Ex: kubectl-commands.md

Troubleshooting : YYYY-MM-DD-[issue].md
  Ex: 2025-12-23-k8s-crashloop.md

Projects : [project]/[type].md
  Ex: transcendance/architecture.md
```

---

## 🔄 Workflow de Note

### 1. Apprendre nouveau concept

```
1. Discovery avec Claude (30 min)
   └─ Conversation, pas de notes pendant

2. Créer note (20 min)
   └─ concepts/[techno]/[concept].md
   └─ Utiliser template
   └─ Écrire avec vos mots

3. Practice autonome (2h)
   └─ Lab, tests

4. Documenter learnings (15 min)
   └─ Pièges vécus
   └─ Liens avec autres concepts
```

### 2. Résoudre problème

```
1. Créer troubleshooting/[date]-[issue].md

2. Documenter en résolvant
   └─ Symptômes
   └─ Tentatives
   └─ Solution

3. Extraire learnings
   └─ Mettre à jour concept lié
   └─ Ajouter à common-errors.md si récurrent
```

### 3. Projet

```
projects/[project-name]/
├── architecture.md     # Design + diagrammes
├── decisions.md        # ADR (pourquoi ces choix)
└── retrospective.md    # Post-projet learnings
```

---

## 🔧 Plugins Obsidian Essentiels

### Must-have (4 plugins)
- **Templater** : Templates dynamiques
- **Git** : Auto-commit + sync
- **Dataview** : Requêtes sur notes
- **Excalidraw** : Diagrammes

### Installation
```
1. Settings → Community plugins
2. Browse → Search plugin
3. Install + Enable
```

### Config Git plugin
```json
{
  "autoCommitMessage": "vault backup: {{date}}",
  "autoPullInterval": 5,
  "autoPushInterval": 5
}
```

---

## 🔄 Intégration Claude Projects

### Sync bidirectionnel

```
OBSIDIAN (source de vérité)
    ↓ Upload sélectif
CLAUDE KB (Project)
    ↓ Nouvelles notes
OBSIDIAN (update)
```

### Quoi uploader où

**Project "DevOps Learning"**
```
Upload depuis Obsidian :
✅ roadmap.md
✅ 5-10 concepts clés référencés souvent
❌ Pas les cheatsheets (consultez directement)
```

**Project "Infrastructure - [Projet]"**
```
Upload depuis Obsidian :
✅ projects/[projet]/architecture.md
✅ projects/[projet]/decisions.md
❌ Pas le code (paste au besoin dans chat)
```

### Quand re-upload ?
```
Changement MAJEUR uniquement :
✅ Nouvelle section architecture
✅ Décision technique importante
❌ Typos, petits ajouts (attendre batch mensuel)
```

---

## 📊 Requêtes Dataview Utiles

### Concepts à réviser
```dataview
TABLE difficulty, last-reviewed
FROM "concepts"
WHERE contains(tags, "status/review")
SORT last-reviewed ASC
```

### Projets actifs
```dataview
LIST
FROM "projects"
WHERE contains(tags, "active")
SORT updated DESC
```

### Troubleshooting récent
```dataview
TABLE file.ctime as "Date"
FROM "troubleshooting"
WHERE file.ctime > date(today) - dur(7 days)
```

---

## 🔄 Maintenance

### Quotidien (5 min)
- Commit Git des modifs
- Daily note si utilisé

### Hebdomadaire (30 min)
- Review notes status/learning → mastered
- Update liens cassés
- Push Git

### Mensuel (1-2h)
- Review graph (notes orphelines ?)
- Consolidation notes similaires
- Nettoyage attachments
- Re-upload dans Claude KB si changements majeurs

---

## ✅ Checklist Setup

### Installation
- [ ] Installer Obsidian
- [ ] Créer vault ~/devops-vault/
- [ ] Créer structure dossiers
- [ ] Installer plugins (Templater, Git, Dataview, Excalidraw)
- [ ] Copier templates

### Configuration
- [ ] Config Git (init, remote, auto-commit)
- [ ] Config Templater (folder templates)
- [ ] Activer Dataview

### Premier contenu
- [ ] Import roadmap
- [ ] Créer première note concept
- [ ] Tester liens [[bidirectionnels]]
- [ ] Premier commit Git

### Validation
- [ ] Graph view affiche liens
- [ ] Git commit fonctionne
- [ ] Templates fonctionnent

---

**Setup time : 30 min**
**Maintenance : 30 min/semaine**
**ROI : Massive (knowledge capitalisé à vie)**