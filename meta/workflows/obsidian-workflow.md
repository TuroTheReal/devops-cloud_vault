# 🔄 Obsidian Daily Workflow

## 🎯 Objectif

Workflow quotidien pour maximiser rétention et organisation avec Obsidian.

**Prerequisites**: [[obsidian-setup]] completé

---

## 🧠 Pourquoi ce Workflow?

Obsidian transforme ton vault en **Personal Knowledge Management (PKM)** avec:
- **Liens bidirectionnels** : Vois toutes les notes qui référencent un concept
- **Graph view** : Visualise connexions entre technologies
- **Recherche puissante** : Trouve info en <30s dans 100+ notes
- **Tags & filtres** : Organise par statut, difficulté, technologie

---

## 📖 Atomisation des Notes : Règle d'Or

### ❓ Une Note = Un Concept Maîtrisable en 1 Session

**Principe** : **"1 Note = 1 Idée Maîtrisable en 5-10 minutes"**

### ✅ Créer Note Séparée Si :

1. **Concept réutilisable** : Utilisé dans multiple contextes
   - Exemple: "Healthchecks" dans Docker, Compose, Swarm
2. **Temps d'apprentissage > 2h** : Assez substantiel
3. **Peut référencer dans plusieurs projets** : "Overlay networks" dans Glasck + futurs
4. **A ses propres pitfalls** : Assez de contenu pour section dédiée
5. **Complexité 3+/5** : Mérite explication approfondie

### ❌ Garder dans Même Note Si :

1. **Trop petit** : Un seul commande/flag → va dans cheatsheet
2. **Trop lié** : "FROM instruction" séparé de "Dockerfile" = inutile
3. **Pas réutilisable** : Spécifique à un projet → va dans `projects/`
4. **< 30min pour maîtriser** : Juste une section dans note parente

---

## 🎓 Workflow : Apprendre Nouveau Concept

### Phase 1: Découverte (30-60 min)

```
1. Conversation avec Claude
   ├─ Pose questions
   ├─ Demande exemples
   └─ NE PAS prendre notes pendant (focus compréhension)

2. Créer note après compréhension
   ├─ concepts/technology/concept-name.md
   ├─ Utiliser template concept.md
   └─ Tag: status/discovering
```

**Output** : Note avec TL;DR + When to Use remplis

---

### Phase 2: Apprentissage Actif (2-4h)

```
1. Remplir Key Concepts
   ├─ AVEC TES MOTS (pas copier-coller!)
   ├─ Expliquer comme à un ami
   └─ Tag: status/learning

2. Créer Minimal Example
   ├─ Code testé qui fonctionne
   ├─ Contexte du problème résolu
   └─ Commentaires explicatifs

3. Lier aux concepts connexes
   ├─ Prerequisites: [[prerequisite-1]]
   ├─ Related to: [[related-1]]
   └─ Graph view se construit automatiquement
```

**Output** : Note complète avec exemples fonctionnels

---

### Phase 3: Application (2-8h)

```
1. Appliquer dans vrai projet
   ├─ Créer projects/YYYY-MM-project-name/
   ├─ Documenter en temps réel
   └─ Tag concept: status/practiced

2. Documenter Pitfalls
   ├─ Symptômes rencontrés
   ├─ Ce que tu as essayé (wrong)
   ├─ Solution qui a fonctionné (correct)
   └─ Temps perdu + Lesson learned

3. Créer liens bidirectionnels
   ├─ Dans concept: "Utilisé dans [[project-glasck]]"
   └─ Dans projet: "Utilise [[docker-swarm-overlay-networks]]"
```

**Output** : Concept appliqué + pitfalls documentés + projet lié

---

### Phase 4: Maîtrise (30-60 min)

```
1. Compléter Mastery Checklist
   ├─ Peux expliquer sans notes? ✅
   ├─ Connais use cases? ✅
   ├─ Debuggé erreurs communes? ✅
   └─ Tag: status/mastered

2. Extraire vers cheatsheet
   ├─ Commandes fréquentes
   ├─ Flags importants
   └─ Debugging tips

3. Planifier révision
   ├─ next-review: +1 mois
   ├─ retention-check-1: +7 jours
   └─ retention-check-2: +30 jours
```

**Output** : Concept maîtrisé + cheatsheet + révisions planifiées

---

## 🔧 Workflow : Nouveau Projet

### Pendant le Projet

```
1. Créer structure
   projects/YYYY-MM-project-name/
   └── learnings.md (template project.md)

2. Documenter en temps réel
   ├─ Challenges rencontrés
   ├─ Solutions trouvées
   ├─ Temps investi par phase
   └─ Technologies utilisées

3. Lier aux concepts
   [[docker-swarm-overlay-networks]] utilisé ici
   [[traefik-swarm-integration]] maîtrisé
   [[docker-network-isolation]] appliqué
```

---

### Après le Projet

```
1. Compléter learning report
   ├─ Architecture finale
   ├─ Key learnings (3-5 points)
   └─ Metrics (time, complexity)

2. Extraire nouveaux concepts
   ├─ Si pattern réutilisable → créer concept
   ├─ Si spécifique projet → garder dans project
   └─ Utiliser règles d'atomisation

3. Mettre à jour cheatsheets
   ├─ Nouvelles commandes utiles
   └─ Debugging tips découverts

4. Graph check
   ├─ Ouvrir graph view
   ├─ Vérifier connexions logiques
   └─ Trouver notes orphelines
```

---

## 🔄 Workflow : Résoudre Problème

### Pendant Debugging

```
1. Créer troubleshooting/YYYY-MM-DD-issue-name.md
   ├─ Tag: troubleshooting, TECHNOLOGY
   └─ Documenter en résolvant

2. Structure
   ├─ Symptômes observés
   ├─ Hypothèses testées
   ├─ Tentatives échouées (avec raison)
   ├─ Solution qui a fonctionné
   └─ Temps total debug
```

---

### Après Résolution

```
1. Extraire learnings
   ├─ Mettre à jour concept lié
   ├─ Ajouter pitfall dans concept
   └─ Si récurrent → créer concept dédié

2. Lier aux concepts
   Utilisait [[docker-swarm-deployment-strategies]]
   Solution: [[docker-swarm-healthchecks]]
```

---

## 🔍 Workflow : Retrouver Info Rapidement

### Scénario 1: "Comment faire X?"

```
1. Ctrl+O → Quick switcher
2. Tape mot-clé: "health"
3. Choix: docker-compose-healthchecks.md
4. Section spécifique
5. ✅ Trouvé en 10 secondes
```

---

### Scénario 2: "Quel projet a utilisé X?"

```
1. Ouvre concept [[traefik-swarm-integration]]
2. Scroll en bas → Backlinks panel
3. Vois: project-glasck-deployment
4. Click → Détails du projet
5. ✅ Contexte retrouvé
```

---

### Scénario 3: "Commande pour Y?"

```
1. Ctrl+O → "swarm cheat"
2. [[cheatsheet-docker-swarm]]
3. Ctrl+F → "logs"
4. docker service logs myapp
5. ✅ Commande copiée
```

---

### Scénario 4: "Pourquoi erreur Z?"

```
1. Ctrl+Shift+F → "hang" (global search)
2. Résultat: docker-swarm-overlay-networks
3. Section: Pitfall - MTU fragmentation
4. Solution: MTU 1450
5. ✅ Bug fixé
```

---

## 🔗 Système de Liens

### 1. Liens Hiérarchiques (Dépendances)

```markdown
**Prerequisites**: [[docker-basics]], [[docker-images-layers]]
**Related to**: [[kubernetes-basics]], [[docker-swarm-basics]]
```

**Usage** : Graph view montre learning path

---

### 2. Liens Conceptuels (Connexions Logiques)

```markdown
## Docker Swarm utilise

- [[docker-images-layers]] pour déployer services
- [[docker-network-isolation]] pour sécuriser communications
- [[docker-volumes]] pour persistance données
```

**Usage** : Comprendre architecture complète

---

### 3. Liens de Référence (Pratique)

```markdown
Voir aussi:
- [[cheatsheet-docker-swarm]] pour commandes rapides
- [[project-glasck-deployment]] pour exemple réel
```

**Usage** : Navigation rapide vers ressources pratiques

---

## 📊 Maintenance Quotidienne

### Matin (5 min)

```
1. Daily note (optionnel)
   ├─ Objectif du jour
   └─ Concepts à pratiquer

2. Check révisions
   ├─ Search: next-review < today
   └─ Réviser 1-2 concepts
```

---

### Pendant Travail

```
1. Documenter en temps réel
   ├─ Pitfalls rencontrés
   ├─ Solutions trouvées
   └─ Temps investi

2. Créer liens au fur et à mesure
   ├─ Concept → Projet
   ├─ Concept → Concept
   └─ Graph view se construit
```

---

### Fin de Journée (5 min)

```
1. Update notes
   ├─ Compléter sections en cours
   └─ Update tags si statut changé

2. Git commit (auto si plugin)
   ├─ Changes sauvegardées
   └─ Sync entre machines
```

---

## 📅 Maintenance Hebdomadaire

### Vendredi Après-Midi (30 min)

```
1. Review statuses
   ├─ Search: tag:#status/learning
   ├─ Quels concepts maîtrisés?
   └─ Update tags → status/mastered

2. Graph check
   ├─ Ctrl+G → Graph view
   ├─ Notes orphelines?
   └─ Connexions manquantes?

3. Update cheatsheets
   ├─ Nouvelles commandes apprises?
   └─ Ajouter aux cheatsheets
```

---

## 🗓️ Maintenance Mensuelle

### Dernier Jour du Mois (1-2h)

```
1. Review global
   ├─ Dataview: status/learning vs mastered
   ├─ Technologies progress
   └─ Projets completed

2. Consolidation
   ├─ Notes similaires à merger?
   ├─ Concepts trop fragmentés?
   └─ Cleanup notes obsolètes

3. Graph cleanup
   ├─ Notes orphelines importantes?
   ├─ Créer MOC si nécessaire
   └─ Fix broken links

4. Metrics update
   ├─ Dashboard update
   ├─ Time invested review
   └─ Next month goals
```

---

## 📊 Map of Content (MOC)

Si ton vault dépasse 50+ notes, crée des MOCs (index par technologie).

### Exemple: MOC Docker

**File**: `meta/moc-docker.md`

```markdown
# 📁 Docker - Map of Content

## Concepts Fondamentaux
- [[docker-images-layers]] ⭐⭐⭐ (mastered)
- [[docker-containers-lifecycle]] ⭐⭐ (mastered)
- [[docker-network-isolation]] ⭐⭐⭐ (mastered)

## Orchestration
- [[docker-compose-basics]] ⭐⭐ (mastered)
- [[docker-swarm-overlay-networks]] ⭐⭐⭐⭐ (mastered)
- [[docker-swarm-deployment-strategies]] ⭐⭐⭐ (mastered)

## Projets Utilisant Docker
- [[project-transcendence]] (monitoring focus)
- [[project-glasck-deployment]] (swarm focus)

## Cheatsheets
- [[cheatsheet-docker]]
- [[cheatsheet-docker-compose]]
- [[cheatsheet-docker-swarm]]

## Troubleshooting Commun
- MTU fragmentation → [[docker-swarm-overlay-networks#pitfall-1]]
- OOM kills → [[docker-containers-lifecycle#pitfall-2]]
- Healthcheck failures → [[docker-compose-healthchecks#pitfalls]]
```

---

## 🎯 Système de Révision Espacée

### Rétention Long-Terme

```yaml
# Dans metadata de concept
created: 2025-12-23
next-review: 2026-01-23          # +1 mois
retention-check-1: 2025-12-30    # +7 jours
retention-check-2: 2026-01-23    # +30 jours
retention-check-3: 2026-04-23    # +3 mois
```

### Workflow de Révision

```
Day +7 (retention-check-1):
├─ Relis note sans regarder code
├─ Peux expliquer concept? ✅
├─ Peux lister use cases? ✅
└─ Update tag si nécessaire

Day +30 (retention-check-2):
├─ Explique concept à voix haute (sans notes!)
├─ Liste pitfalls de mémoire
└─ Recherche commandes dans cheatsheet

Day +90 (retention-check-3):
├─ Applique dans nouveau projet
├─ Vérifier si info manquante
└─ Update concept avec nouvelles learnings
```

---

## 🔍 Recherche Avancée

### Syntaxe Obsidian Search

```
# Tous les concepts Docker
path:concepts/docker

# Notes non maîtrisées
tag:#status/learning

# Pitfalls documentés
"Pitfall" path:concepts

# Projets avec Swarm
tag:#swarm path:projects

# Notes créées cette semaine
created:7d

# Combinaisons
path:concepts tag:#docker tag:#status/mastered
```

---

## 📊 Dashboard Personnel (Optionnel)

**File**: `meta/dashboard.md`

```markdown
# 📊 DevOps Knowledge Dashboard

## 📈 Statistiques

### Concepts
- **Total**: 9
- **Mastered**: 6 ✅
- **Learning**: 2 🟡
- **To Learn**: 1 📝

### Technologies
- **Docker**: ⭐⭐⭐⭐ (4/5) - 35h invested
- **Traefik**: ⭐⭐⭐⭐ (4/5) - 20h invested
- **Kubernetes**: ⭐⭐ (2/5) - 8h invested

### Projets
- **Completed**: 2 (Transcendence, Glasck)
- **In Progress**: 0
- **Planned**: 1 (Multi-node Swarm)

---

## 🎯 Cette Semaine

### Apprendre
- [ ] Réviser [[docker-swarm-overlay-networks]]
- [ ] Finir [[kubernetes-basics]]

### Appliquer
- [ ] Projet: Deploy multi-node Swarm cluster

### Révisions Planifiées
\`\`\`dataview
TABLE next-review
FROM "concepts"
WHERE next-review <= date(today) + dur(7 days)
SORT next-review ASC
\`\`\`

---

## 📚 En Cours

\`\`\`dataview
LIST
FROM "concepts"
WHERE contains(tags, "status/learning")
\`\`\`

---

**Last update**: {{date}}
```

---

## ✅ Checklist : Bon Workflow

### Daily
- [ ] Documenter learnings en temps réel
- [ ] Créer liens entre concepts
- [ ] Update tags si statut changé
- [ ] Git commit (auto ou manuel)

### Weekly
- [ ] Review status/learning → mastered
- [ ] Graph view check
- [ ] Update cheatsheets
- [ ] Plan prochaine semaine

### Monthly
- [ ] Dataview metrics review
- [ ] Consolidation notes similaires
- [ ] Graph cleanup (orphelines)
- [ ] Dashboard update

---

## 🎓 Résumé : Les 3 Règles d'Or

### 1. **1 Note = 1 Concept Maîtrisable**
- Peux-tu l'expliquer en 5-10 min? ✅
- Est-ce réutilisable? ✅
- → Alors c'est une bonne note atomique

### 2. **Liens > Hiérarchie**
- Ne te perds pas dans structure dossiers complexe
- Utilise [[liens]] pour connexions
- Graph view montrera organisation naturelle

### 3. **Simple > Complexe**
- Commence minimal
- Ajoute complexité si besoin (MOC, Dataview)
- Objectif = retrouver info en <30s

---

## 📖 Exemples Réels

### Bien Atomisé (Ton Vault Actuel ✅)

```
docker/
├── docker-images-layers.md         # 1 concept = image optimization
├── docker-containers-lifecycle.md  # 1 concept = container states
├── docker-network-isolation.md     # 1 concept = network security
└── docker-swarm-overlay-networks.md # 1 concept = Swarm networking

Pourquoi? Chaque note = concept maîtrisable séparément,
réutilisable dans différents projets.
```

### Trop Fragmenté (À Éviter ❌)

```
docker/
├── docker-image-layers.md
├── docker-image-caching.md      # Trop séparé!
├── docker-image-buildkit.md     # Même concept
└── docker-multi-stage-1.md      # Trop fragmenté
```

### Pas Assez Atomisé (À Éviter ❌)

```
docker/
└── docker-complete-guide.md  # 10,000 lignes! Ingérable
```

---

**Time investment**: 10-20 min/jour pour workflow optimal

**ROI**: Knowledge capitalisé à vie, retrieval <30s

**Next**: Start with templates in [[obsidian-setup]]

---

**Last update**: 2025-12-23
