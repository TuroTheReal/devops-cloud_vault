# 🔧 Obsidian Setup Guide

## 🎯 Goal

Configure Obsidian to transform your markdown vault into an interconnected knowledge base.

---

## 📁 Vault Structure

```
devops-cloud_vault/
│
├── concepts/              # Theory (how it works)
│   ├── docker/
│   ├── traefik/
│   └── monitoring/
│
├── cheatsheets/           # Practice (quick commands)
│   ├── docker/
│   ├── traefik/
│   └── taskfile/
│
├── projects/              # Experience (what you did)
│   └── YYYY-MM-project-name/
│       └── YYYY-MM-project-name.md
│
├── troubleshooting/       # Problems & solutions
│
└── meta/                  # Metadata & guides
    ├── templates/
    ├── guides/
    └── workflows/
```

---

## 🔧 Installation

### 1. Install Obsidian
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

### 2. Open the Vault
```
File → Open vault → Open folder as vault
Select: ~/abGitHub/devops-cloud_vault
```

---

## 🎨 Basic Configuration

### Core Plugins (Enabled by Default)
```
Settings → Core Plugins
✅ Graph view           # Visualize connections
✅ Backlinks            # See reverse references
✅ Quick switcher       # Ctrl+O quick navigation
✅ Search               # Ctrl+Shift+F global search
✅ Templates            # Use meta/templates/
```

### Recommended Settings
```
Settings → Editor
✅ Strict line breaks
✅ Show line number
✅ Readable line length: OFF (for code blocks)

Settings → Files & Links
✅ Default location for new notes: Same folder as current file
✅ New link format: Shortest path
✅ Use [[Wikilinks]]: ON
```

---

## 📦 Essential Plugins

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

#### 2. **Dataview** (Dynamic queries)
```
Search: "Dataview"
Install + Enable

Lets you create views like:
- All concepts with status/learning
- Projects by technology
- Notes to review
```

#### 3. **Advanced Tables** (Table editing)
```
Search: "Advanced Tables"
Install + Enable

Makes markdown table editing easier
```

---

## 🏷️ Tag System

### Status Tags
```yaml
tags: [status/discovering]   # Initial discovery
tags: [status/learning]      # Learning in progress
tags: [status/practiced]     # Applied in project
tags: [status/mastered]      # Mastered
tags: [status/review]        # Needs review
```

### Type Tags
```yaml
tags: [concept, docker]
tags: [cheatsheet, traefik]
tags: [project, swarm]
```

### Difficulty Tags
```yaml
difficulty: ⭐ (1/5)     # Basic
difficulty: ⭐⭐⭐ (3/5)  # Intermediate
difficulty: ⭐⭐⭐⭐⭐ (5/5) # Expert
```

### Complete Example
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

## 📊 Dataview - Useful Queries

### Concepts to Review
```dataview
TABLE difficulty, time-to-master, next-review
FROM "concepts"
WHERE contains(tags, "status/learning")
SORT next-review ASC
```

### Projects by Technology
```dataview
LIST
FROM "projects"
WHERE contains(tags, "docker-swarm")
SORT file.ctime DESC
```

### Recent Troubleshooting
```dataview
TABLE file.ctime as "Date"
FROM "troubleshooting"
WHERE file.ctime > date(today) - dur(7 days)
SORT file.ctime DESC
```

### Mastered Technologies
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

# If not already done
git init
git remote add origin <your-repo-url>

# First commit
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

### Open Graph View
```
Ctrl+G or View → Graph view
```

### Useful Filters
```
# Show only concepts
path:concepts

# Show only Docker
tag:#docker

# Exclude cheatsheets
-path:cheatsheets

# Only mastered notes
tag:#status/mastered
```

### Color Groups
```
Settings → Graph view → Groups

Group 1: tag:#docker → Blue
Group 2: tag:#traefik → Green
Group 3: tag:#monitoring → Orange
Group 4: tag:#status/learning → Red
```

---

## 🗂️ Templates

### Create Template Folder
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
- [[traefik-integration]]
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

## 🔍 Essential Obsidian Shortcuts

```
Ctrl+N         : New note
Ctrl+O         : Quick switcher (open note by name)
Ctrl+P         : Command palette
Ctrl+Shift+F   : Global search
Ctrl+G         : Graph view
Ctrl+Click     : Open link in new pane
Ctrl+E         : Toggle edit/preview
[[             : Create link (auto-complete)
Ctrl+K         : Insert link
```

---

## ✅ Setup Checklist

### Installation
- [ ] Install Obsidian
- [ ] Open existing vault
- [ ] Enable core plugins (Graph, Backlinks, Templates)
- [ ] Install community plugins (Git, Dataview)

### Configuration
- [ ] Configure Git plugin (auto-commit)
- [ ] Configure Templates folder
- [ ] Configure Graph view groups
- [ ] Test search and navigation

### Validation
- [ ] Graph view displays connections
- [ ] Git auto-commit works
- [ ] Templates available
- [ ] Search works quickly

---

**Setup time**: 30 minutes
**Next**: See [[obsidian-workflow]] for daily usage

---

**Last update**: 2025-12-23
