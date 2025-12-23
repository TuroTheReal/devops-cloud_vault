# [Outil] - Cheatsheet

## 📋 Métadonnées

```yaml
tags: [cheatsheet, [techno]]
created: YYYY-MM-DD
version: X.Y.Z
```

**Doc officielle** : [URL]
**Concepts liés** : [[concept-1]]

---

## 🚀 Installation

```bash
# macOS
brew install [outil]

# Ubuntu
apt install [outil]

# Vérification
[outil] --version
```

---

## 📚 Commandes Essentielles

### Liste / Get
```bash
# Lister tout
[outil] get [resource]

# Avec détails
[outil] get [resource] -o wide

# Format YAML
[outil] get [resource] -o yaml
```

### Créer
```bash
# Depuis fichier
[outil] create -f file.yaml

# Impératif
[outil] create [resource] [name] [options]
```

### Modifier
```bash
# Appliquer changements
[outil] apply -f file.yaml

# Éditer interactif
[outil] edit [resource] [name]
```

### Supprimer
```bash
# Par nom
[outil] delete [resource] [name]

# Par selector
[outil] delete [resource] -l [label]
```

---

## 🔍 Debug / Inspection

### Logs
```bash
# Voir logs
[outil] logs [resource]

# Follow logs
[outil] logs [resource] -f

# Filtrer
[outil] logs [resource] | grep ERROR
```

### Describe / Info
```bash
# Détails complets
[outil] describe [resource] [name]

# Status
[outil] get [resource] [name] -o wide
```

### Shell / Exec
```bash
# Shell dans ressource
[outil] exec -it [resource] -- /bin/bash

# Commande one-shot
[outil] exec [resource] -- [command]
```

---

## 🎨 Formatting

### Output Formats
```bash
# Wide
[outil] get [resource] -o wide

# JSON
[outil] get [resource] -o json

# Custom columns
[outil] get [resource] -o custom-columns=NAME:.metadata.name
```

### Filtering
```bash
# Par label
[outil] get [resource] -l key=value

# Par field
[outil] get [resource] --field-selector status=Running
```

---

## 🎯 One-Liners Utiles

```bash
# [Description 1]
[commande complexe 1]

# [Description 2]
[commande avec pipe 2]

# [Description 3]
[commande loop 3]
```

---

## 🆘 Troubleshooting

### Problème 1 : [Symptôme]
```bash
# Diagnostic
[commande check]

# Fix
[commande fix]
```

### Problème 2 : [Symptôme]
```bash
# Check + Fix
[commandes]
```

---

## 📝 Config File

```yaml
# [Description fichier]
apiVersion: [version]
kind: [type]
metadata:
  name: [name]
spec:
  # Configuration
  [field]: [value]
```

**Champs obligatoires** :
- `field1` : [Description]
- `field2` : [Description]

---

## 🔗 Aliases

```bash
# ~/.bashrc ou ~/.zshrc

alias k='[outil]'
alias kg='[outil] get'
alias kd='[outil] describe'
alias kl='[outil] logs'

# Fonction utile
function [name]() {
    [commandes avec $1, $2]
}
```

---

## ✅ Best Practices

### À FAIRE
- **[Practice 1]** : [Pourquoi]
  ```bash
  # ✅ Bon
  [commande]
  ```

- **[Practice 2]** : [Pourquoi]

### À ÉVITER
- **[Anti-pattern 1]** : [Pourquoi mal]
  ```bash
  # ❌ Mauvais
  [commande dangereuse]

  # ✅ Bon
  [commande safe]
  ```

---

## 📖 Ressources

- [Official Docs](URL)
- [[concept-fondamental]]
- [[troubleshooting-[outil]]]

---

## 💡 Tips Personnels

### Workflow quotidien
```bash
# Routine typique
[commandes que j'utilise souvent]
```

### Erreurs faites
- **[Erreur 1]** → Fix : [Solution]
- **[Erreur 2]** → Fix : [Solution]

---

## 📌 Quick Reference Card

```
┌────────────────────────────────────┐
│     [OUTIL] - ESSENTIALS           │
├────────────────────────────────────┤
│ LIST       [commande]              │
│ CREATE     [commande]              │
│ UPDATE     [commande]              │
│ DELETE     [commande]              │
│ LOGS       [commande]              │
│ DEBUG      [commande]              │
│                                    │
│ Flags:                             │
│   -o [format]                      │
│   -l [label]                       │
│   -f [file]                        │
└────────────────────────────────────┘
```

---

**Dernière update** : YYYY-MM-DD
**Version outil** : X.Y.Z