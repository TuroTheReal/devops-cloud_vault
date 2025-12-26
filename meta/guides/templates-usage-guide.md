# Templates Usage Guide - Quick Reference

## 🎯 TL;DR

**Temps rédaction optimisé** :
- Concept : **30-40 min** (vs 1h30 avant)
- Project : **45-60 min** (vs 2h-3h avant)
- Cheatsheet : **20-30 min** (vs 1h-1h30 avant)

**Gain global** : **~65% temps économisé** 🚀

---

## 📝 Template à utiliser ?

### Pour un CONCEPT

Toujours utiliser : [meta/templates/concept-template.md](../templates/concept-template.md)

**Quand l'utiliser** :
- Après un lab pratique
- Extraction depuis un projet
- Concept appris et testé

**Sections essentielles (VOS MOTS)** :
1. ✅ **TL;DR + Analogy** (5 min) - Reformulation ultra-concise
2. ✅ **Key Concepts - My understanding** (15-20 min) - 3-5 concepts max
3. ✅ **Minimal Example** (10 min) - Code testé
4. ✅ **Pitfalls Experienced** (10 min) - Seulement si vécu

**Temps total** : 30-40 min

---

### Pour un PROJECT

Toujours utiliser : [meta/templates/project-template.md](../templates/project-template.md)

**Quand l'utiliser** :
- À la fin d'un projet/lab
- Post-mortem extraction

**Sections essentielles** :
1. ✅ **What I Learned** (15 min) - 3-5 concepts max
2. ✅ **Challenges & Solutions** (20 min) - Pitfalls vécus
3. ✅ **Time Breakdown** (5 min) - Tracking 70/30

**Temps total** : 45-60 min

---

### Pour un CHEATSHEET

Toujours utiliser : [meta/templates/cheatsheet-template.md](../templates/cheatsheet-template.md)

**Quand l'utiliser** :
- Commandes utilisées fréquemment
- Extraction depuis projets

**Sections essentielles** :
1. ✅ **Quick Start** (5 min)
2. ✅ **Commands by Operation** (10 min)
3. ✅ **Troubleshooting** (5 min) - Erreurs vécues

**Temps total** : 20-30 min

---

## 🚀 Workflow Optimisé

### Après un lab/projet

**Exemple** : VPS Hetzner SSH hardening (2h30)

```
1. Remplir project learnings (45 min)
   projects/2025-12-vps-hetzner/learnings.md

2. Extraire concepts (3 × 15 min = 45 min)
   concepts/linux/linux-ssh-hardening.md
   concepts/linux/linux-firewall-ufw.md
   concepts/linux/linux-systemd-socket.md

3. Update cheatsheet optionnel (15 min)
   cheatsheets/linux/security-admin-commands.md

Total documentation: 1h30 ✅
```

---

## ✅ Checklist Rapide

### Concept complet ?

- [ ] TL;DR + Analogy (simple et clair)
- [ ] When to Use (2 good + 1 bad)
- [ ] Key Concepts (3-5 max, VOS MOTS)
- [ ] Minimal Example (code testé)
- [ ] Pitfalls (seulement si vécu)
- [ ] Stats (temps + status)
- [ ] Links Obsidian (prerequisites, related)

**Temps** : 30-40 min

---

### Project complet ?

- [ ] Metadata (tags, duration, status)
- [ ] Context (objective, architecture, stack)
- [ ] What I Learned (3-5 concepts)
- [ ] Challenges & Solutions (2-4 max)
- [ ] Time Breakdown (tracking 70/30)
- [ ] Extractions TODO (liens concepts)

**Temps** : 45-60 min

---

### Cheatsheet complet ?

- [ ] Quick Start (install + basic usage)
- [ ] Commands by Operation (CRUD)
- [ ] Debug / Inspection
- [ ] Troubleshooting (erreurs vécues)
- [ ] Personal Notes (workflow quotidien)

**Temps** : 20-30 min

---

## 💡 Tips

### Accélérer la rédaction

1. **Capturer à chaud** : Documenter PENDANT le lab, pas après
2. **Brouillon OK** : Vaut mieux note rapide qu'absence de note
3. **Max 3-5 concepts** : Focus sur l'essentiel
4. **Pitfalls = Or** : Prioriser les erreurs vécues
5. **Links Obsidian** : Toujours lier aux concepts/projets connexes

### Workflow documentation

```
Pendant le lab:
→ Noter erreurs/solutions dans fichier brouillon

Après le lab:
→ 45-60 min remplir project-template
→ 3 × 30 min extraire concepts
→ Total: ~2h doc max
```

### Obsidian Links

**Format** : `[[file-name]]` ou `[[folder/file-name]]`

**Exemples** :
- `[[docker-containers-lifecycle]]`
- `[[concepts/docker/docker-networking]]`
- `[[cheatsheets/docker/docker]]`
- `[[projects/2025-12-vps-hetzner-init-setup/learnings]]`

**Bénéfices** :
- Graph view visualise connexions
- Backlinks montre où concept est utilisé
- Navigation rapide Ctrl+Click

---

## 📊 Gain Temps

| Activité | Avant | Après | Gain |
|----------|-------|-------|------|
| Concept extraction | 1h30 | 30-40 min | **-65%** |
| Project doc | 2h-3h | 45-60 min | **-70%** |
| Cheatsheet | 1h-1h30 | 20-30 min | **-70%** |
| Post-projet VPS | 7h30 | 1h30 | **-80%** |

**Sur Phase 1 Linux (60h)** :
- Documentation avant : 15h
- Documentation après : **4h**
- **Gain : 11h** ✅

**Sur roadmap complet (1350h)** :
- Documentation avant : 200h
- Documentation après : **60h**
- **Gain : 140h = 2 semaines complètes** 🎉

---

## 🎓 Philosophie

### Accepter l'imperfection

```diff
- ❌ Note parfaite jamais écrite
+ ✅ Brouillon utilisable capturé à chaud

- ❌ Toutes les sections remplies
+ ✅ Sections essentielles (VOS MOTS)

- ❌ Paralysie perfectionniste
+ ✅ Documentation progressive
```

### Focus mémorisation

**Sections haute valeur** (vos mots) :
1. TL;DR + Analogy
2. Key Concepts - My understanding
3. Minimal Example
4. Pitfalls Experienced

**Sections basse valeur** (supprimées) :
- Essential Commands (→ cheatsheet)
- Tests Done (rigide)
- Resources externes (peu utilisé)
- Timeline détaillé (chronophage)

---

**Last update**: 2025-12-26
