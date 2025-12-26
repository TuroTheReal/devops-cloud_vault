# 🔧 Obsidian Setup Guide

## 🎯 Objectif

Configurer Obsidian pour transformer ton vault markdown en base de connaissances interconnectée.

---

## 📁 Structure du Vault

```
devops-cloud_vault/
│
├── concepts/              # Théorie (comment ça marche)
│   ├── docker/
│   ├── traefik/
│   └── monitoring/
│
├── cheatsheets/           # Pratique (commandes rapides)
│   ├── docker/
│   ├── traefik/
│   └── taskfile/
│
├── projects/              # Expérience (ce que tu as fait)
│   └── YYYY-MM-project-name/
│       └── learnings.md
│
├── troubleshooting/       # Problèmes & solutions
│
└── meta/                  # Métadonnées & guides
    ├── templates/
    ├── guides/
    └── workflows/
```

---

## 🔧 Installation

### 1. Installer Obsidian
```bash
# macOS
brew install --cask obsidian

# Linux (AppImage)
wget https://github.com/obsidianmd/obsidian-releases/releases/download/v1.5.3/Obsidian-1.5.3.AppImage
chmod +x Obsidian-1.5.3.AppImage
./Obsidian-1.5.3.AppImage

# Windows
# Download from https://obsidian.md
```

### 2. Ouvrir le Vault
```
File → Open vault → Open folder as vault
Select: ~/abGitHub/devops-cloud_vault
```

---

## 🎨 Configuration de Base

### Core Plugins (Activés par Défaut)
```
Settings → Core Plugins
✅ Graph view           # Visualiser connexions
✅ Backlinks            # Voir références inverses
✅ Quick switcher       # Ctrl+O navigation rapide
✅ Search               # Ctrl+Shift+F recherche globale
✅ Templates            # Utiliser meta/templates/
```

### Paramètres Recommandés
```
Settings → Editor
✅ Strict line breaks
✅ Show line number
✅ Readable line length: OFF (pour code blocks)

Settings → Files & Links
✅ Default location for new notes: Same folder as current file
✅ New link format: Shortest path
✅ Use [[Wikilinks]]: ON
```

---

## 📦 Plugins Essentiels

### Must-Have Plugins

#### 1. **Git** (Auto-backup)
```
Settings → Community Plugins → Browse
Search: "Obsidian Git"
Install + Enable

Configuration:
- Vault backup interval: 10 minutes
- Auto commit message: "vault backup: {{date}}"
- Auto pull interval: 5 minutes
- Auto push interval: 10 minutes
```

#### 2. **Dataview** (Requêtes dynamiques)
```
Search: "Dataview"
Install + Enable

Permet créer vues comme:
- Tous les concepts status/learning
- Projets par technologie
- Notes à réviser
```

#### 3. **Advanced Tables** (Édition tables)
```
Search: "Advanced Tables"
Install + Enable

Facilite édition des tables markdown
```

---

## 🏷️ Système de Tags

### Tags de Statut
```yaml
tags: [status/discovering]   # Découverte initiale
tags: [status/learning]      # En apprentissage
tags: [status/practiced]     # Appliqué dans projet
tags: [status/mastered]      # Maîtrisé
tags: [status/review]        # À réviser
```

### Tags de Type
```yaml
tags: [concept, docker]
tags: [cheatsheet, traefik]
tags: [project, swarm]
```

### Tags de Difficulté
```yaml
difficulty: ⭐ (1/5)     # Basique
difficulty: ⭐⭐⭐ (3/5)  # Intermédiaire
difficulty: ⭐⭐⭐⭐⭐ (5/5) # Expert
```

### Exemple Complet
```yaml
---
tags: [concept, docker, swarm, networking, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐⭐ (4/5)
time-to-master: 8h
next-review: 2026-01-23
---
```

---

## 📊 Dataview - Requêtes Utiles

### Concepts à Réviser
```dataview
TABLE difficulty, time-to-master, next-review
FROM "concepts"
WHERE contains(tags, "status/learning")
SORT next-review ASC
```

### Projets par Technologie
```dataview
LIST
FROM "projects"
WHERE contains(tags, "docker-swarm")
SORT file.ctime DESC
```

### Troubleshooting Récent
```dataview
TABLE file.ctime as "Date"
FROM "troubleshooting"
WHERE file.ctime > date(today) - dur(7 days)
SORT file.ctime DESC
```

### Technologies Maîtrisées
```dataview
TABLE difficulty, time-to-master
FROM "concepts"
WHERE contains(tags, "status/mastered")
SORT difficulty DESC
```

---

## 🔄 Git Configuration

### Initial Setup
```bash
cd ~/abGitHub/devops-cloud_vault

# Si pas déjà fait
git init
git remote add origin <your-repo-url>

# Premier commit
git add .
git commit -m "Initial vault setup"
git push -u origin main
```

### Obsidian Git Plugin Config
```json
{
  "commitMessage": "vault backup: {{date}}",
  "autoCommitMessage": "vault backup: {{date}} {{hostname}}",
  "commitDateFormat": "YYYY-MM-DD HH:mm:ss",
  "autoSaveInterval": 10,
  "autoPullInterval": 5,
  "autoPushInterval": 10,
  "autoPullOnBoot": true,
  "disablePush": false,
  "pullBeforePush": true,
  "disablePopups": false,
  "listChangedFilesInMessageBody": false,
  "showStatusBar": true,
  "updateSubmodules": false
}
```

---

## 🎨 Graph View Configuration

### Ouvrir Graph View
```
Ctrl+G ou View → Graph view
```

### Filtres Utiles
```
# Montrer seulement concepts
path:concepts

# Montrer seulement Docker
tag:#docker

# Exclure cheatsheets
-path:cheatsheets

# Seulement notes maîtrisées
tag:#status/mastered
```

### Groupes de Couleurs
```
Settings → Graph view → Groups

Group 1: tag:#docker → Blue
Group 2: tag:#traefik → Green
Group 3: tag:#monitoring → Orange
Group 4: tag:#status/learning → Red
```

---

## 🗂️ Templates

### Créer Template Folder
```
Settings → Core Plugins → Templates
Template folder location: meta/templates
```

### Template: Concept
**File**: `meta/templates/concept.md`
```markdown
# {{title}}

## 📋 Metadata

\`\`\`yaml
tags: [concept, TECHNOLOGY, status/learning]
created: {{date}}
updated: {{date}}
difficulty: ⭐⭐ (2/5)
time-to-master: Xh
\`\`\`

**Prerequisites**: [[prerequisite-1]], [[prerequisite-2]]
**Related to**: [[related-1]], [[related-2]]

---

## 🎯 TL;DR (30 seconds)

Brief explanation of the concept (2-3 sentences).

---

## 🤔 When to Use?

### ✅ Good for
1. Use case 1
2. Use case 2

### ❌ Bad for
- Anti-pattern 1
- Anti-pattern 2

---

## 📚 Key Concepts

### 1. Concept Name

**My understanding**:
Explain concept in your own words.

**Why important**:
Why this matters.

**Real example**:
\`\`\`yaml
# Code example
\`\`\`

---

## 💻 Minimal Example

### Context
What problem does this solve?

### Code
\`\`\`bash
# Working example
\`\`\`

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Name

**Symptom**:
What went wrong.

**What I did wrong**:
\`\`\`bash
# ❌ Wrong approach
\`\`\`

**Solution**:
\`\`\`bash
# ✅ Correct approach
\`\`\`

**Time wasted**: Xh
**Lesson**: Key takeaway.

---

## 🔧 Essential Commands

\`\`\`bash
# Command 1
# Command 2
\`\`\`

---

## 📊 Learning Timeline

\`\`\`
YYYY-MM-DD: Discovery
YYYY-MM-DD: First practice
YYYY-MM-DD: Mastery
\`\`\`

### Time Invested

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Discovery | 30min | 1h | 1h30 |
| Practice | - | 2h | 2h |
| Debugging | - | 1h | 1h |
| **TOTAL** | **30min (17%)** | **4h (83%)** | **4h30** |

**Ratio**: 17% ✅ (target <30%)

---

## 📝 Status

**Current status**: 🟡 Learning / ✅ Mastered

---

## 🔗 Resources

### Official
- [Documentation](link)

### Personal
- [[cheatsheet-related]]
- [[project-using-this]]

---

## ✅ Mastery Checklist

### Understanding
- [ ] Can explain concept
- [ ] Understand use cases
- [ ] Know when NOT to use

### Application
- [ ] Built minimal example
- [ ] Applied in real project
- [ ] Debugged common errors

### Solidification
- [ ] Complete documentation
- [ ] Extract to cheatsheet
- [ ] Retention test Day+7

**Validation date**: YYYY-MM-DD
**Total time**: Xh

---

**Last update**: {{date}}
**Next review**: {{date:+1M}}
```

### Template: Cheatsheet
**File**: `meta/templates/cheatsheet.md`
```markdown
# {{title}} Cheatsheet

## 📋 Metadata

\`\`\`yaml
tags: [cheatsheet, TECHNOLOGY]
created: {{date}}
updated: {{date}}
\`\`\`

**Related concepts**: [[concept-1]], [[concept-2]]

---

## 🚀 Quick Start

\`\`\`bash
# Most common command
\`\`\`

---

## 📚 Common Operations

### Category 1

\`\`\`bash
# Operation 1
command --flag value

# Operation 2
command --flag value
\`\`\`

---

## 🔍 Debugging

\`\`\`bash
# Check status
# View logs
# Inspect details
\`\`\`

---

## 💡 Pro Tips

- **Tip 1**: Explanation
- **Tip 2**: Explanation

---

**Last update**: {{date}}
```

### Template: Project
**File**: `meta/templates/project.md`
```markdown
# Project: {{title}}

## 📋 Metadata

\`\`\`yaml
tags: [project, TECHNOLOGIES]
start: {{date}}
end: YYYY-MM-DD
status: in-progress / completed
\`\`\`

---

## 🎯 Objectif

What was the goal?

---

## 🏗️ Architecture

\`\`\`
Diagram or description
\`\`\`

---

## 🛠️ Technologies Utilisées

- [[docker-swarm-overlay-networks]]
- [[traefik-swarm-integration]]
- [[technology-3]]

---

## 📊 Challenges & Solutions

### Challenge 1: Name

**Problem**:
Description.

**Solution**:
What fixed it.

**Learnings**:
Key takeaway.

---

## ⏱️ Time Invested

| Phase | Time | Notes |
|-------|------|-------|
| Planning | Xh | Description |
| Implementation | Xh | Description |
| Debugging | Xh | Description |
| Documentation | Xh | Description |
| **TOTAL** | **Xh** | |

---

## 📝 Key Learnings

1. Learning 1
2. Learning 2
3. Learning 3

---

## 🔗 Resources

- [GitHub Repo](link)
- [Documentation](link)

---

**Last update**: {{date}}
```

---

## 🔍 Raccourcis Obsidian Essentiels

```
Ctrl+N         : Nouvelle note
Ctrl+O         : Quick switcher (ouvrir note par nom)
Ctrl+P         : Palette de commandes
Ctrl+Shift+F   : Recherche globale
Ctrl+G         : Graph view
Ctrl+Click     : Ouvrir lien dans nouveau pane
Ctrl+E         : Toggle edit/preview
[[             : Créer lien (auto-complétion)
Ctrl+K         : Insérer lien
```

---

## ✅ Checklist Setup

### Installation
- [ ] Installer Obsidian
- [ ] Ouvrir vault existant
- [ ] Activer core plugins (Graph, Backlinks, Templates)
- [ ] Installer community plugins (Git, Dataview)

### Configuration
- [ ] Configurer Git plugin (auto-commit)
- [ ] Configurer Templates folder
- [ ] Configurer Graph view groups
- [ ] Tester recherche et navigation

### Validation
- [ ] Graph view affiche connexions
- [ ] Git auto-commit fonctionne
- [ ] Templates disponibles
- [ ] Recherche fonctionne rapidement

---

**Setup time**: 30 minutes
**Next**: See [[obsidian-workflow]] for daily usage

---

**Last update**: 2025-12-23
