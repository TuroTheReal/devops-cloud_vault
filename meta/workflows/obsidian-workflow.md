# 🔄 Obsidian Daily Workflow

## 🎯 Goal

Daily workflow to maximize retention and organization with Obsidian.

**Prerequisites**: [[obsidian-setup]] completed

---

## 🧠 Why This Workflow?

Obsidian transforms your vault into **Personal Knowledge Management (PKM)** with:
- **Bidirectional links**: See all notes that reference a concept
- **Graph view**: Visualize connections between technologies
- **Powerful search**: Find info in <30s across 100+ notes
- **Tags & filters**: Organize by status, difficulty, technology

---

## 📖 Note Atomization: Golden Rule

### ❓ One Note = One Concept Masterable in 1 Session

**Principle**: **"1 Note = 1 Idea Masterable in 5-10 minutes"**

### ✅ Create Separate Note If:

1. **Reusable concept**: Used in multiple contexts
   - Example: "Healthchecks" in Docker, Compose, Swarm
2. **Learning time > 2h**: Substantial enough
3. **Can reference in multiple projects**: "Overlay networks" in Glasck + future ones
4. **Has its own pitfalls**: Enough content for dedicated section
5. **Complexity 3+/5**: Deserves thorough explanation

### ❌ Keep in Same Note If:

1. **Too small**: Single command/flag → goes in cheatsheet
2. **Too coupled**: "FROM instruction" separate from "Dockerfile" = useless
3. **Not reusable**: Specific to one project → goes in `projects/`
4. **< 30min to master**: Just a section in parent note

---

## 🎓 Workflow: Learning New Concept

### Phase 1: Discovery (30-60 min)

```
1. Conversation with Claude
   ├─ Ask questions
   ├─ Request examples
   └─ DON'T take notes during (focus on understanding)

2. Create note after understanding
   ├─ concepts/technology/concept-name.md
   ├─ Use concept.md template
   └─ Tag: status/discovering
```

**Output**: Note with TL;DR + When to Use filled in

---

### Phase 2: Active Learning (2-4h)

```
1. Fill in Key Concepts
   ├─ IN YOUR OWN WORDS (don't copy-paste!)
   ├─ Explain like to a friend
   └─ Tag: status/learning

2. Create Minimal Example
   ├─ Tested code that works
   ├─ Problem context solved
   └─ Explanatory comments

3. Link to related concepts
   ├─ Prerequisites: [[prerequisite-1]]
   ├─ Related to: [[related-1]]
   └─ Graph view builds automatically
```

**Output**: Complete note with working examples

---

### Phase 3: Application (2-8h)

```
1. Apply in real project
   ├─ Create projects/YYYY-MM-project-name/
   ├─ Document in real-time
   └─ Tag concept: status/practiced

2. Document Pitfalls
   ├─ Symptoms encountered
   ├─ What you tried (wrong)
   ├─ Solution that worked (correct)
   └─ Time wasted + Lesson learned

3. Create bidirectional links
   ├─ In concept: "Used in [[project-glasck]]"
   └─ In project: "Uses [[docker-swarm-overlay-networks]]"
```

**Output**: Applied concept + documented pitfalls + linked project

---

### Phase 4: Mastery (15-20 min)

```
1. Finalize Stats
   ├─ Total time: Xh (X% assisted / X% autonomous)
   ├─ Status: ✅ Mastered
   └─ Used in: [[project-1]], [[project-2]]

2. Extract to cheatsheet (optional)
   ├─ Frequent commands
   ├─ Discovered debugging tips
   └─ ~20 min with simplified cheatsheet-template

3. Plan next review
   └─ Next review: +1 month
```

**Output**: Mastered concept + stats + optional cheatsheet

**Total concept documentation time**: **30-40 min** ✅

---

## 🔧 Workflow: New Project

### During Project

```
1. Create structure
   projects/YYYY-MM-project-name/
   └── learnings.md (project.md template)

2. Document in real-time
   ├─ Challenges encountered
   ├─ Solutions found
   ├─ Time invested by phase
   └─ Technologies used

3. Link to concepts
   [[docker-swarm-overlay-networks]] used here
   [[traefik-integration]] mastered
   [[docker-network-isolation]] applied
```

---

### After Project

```
1. Complete learning report
   ├─ Final architecture
   ├─ Key learnings (3-5 points)
   └─ Metrics (time, complexity)

2. Extract new concepts
   ├─ If reusable pattern → create concept
   ├─ If project-specific → keep in project
   └─ Use atomization rules

3. Update cheatsheets
   ├─ New useful commands
   └─ Discovered debugging tips

4. Graph check
   ├─ Open graph view
   ├─ Verify logical connections
   └─ Find orphan notes
```

---

## 🔄 Workflow: Solving Problem

### During Debugging

```
1. Create troubleshooting/YYYY-MM-DD-issue-name.md
   ├─ Tag: troubleshooting, TECHNOLOGY
   └─ Document while solving

2. Structure
   ├─ Observed symptoms
   ├─ Tested hypotheses
   ├─ Failed attempts (with reason)
   ├─ Solution that worked
   └─ Total debug time
```

---

### After Resolution

```
1. Extract learnings
   ├─ Update related concept
   ├─ Add pitfall in concept
   └─ If recurring → create dedicated concept

2. Link to concepts
   Was using [[docker-swarm-deployment-strategies]]
   Solution: [[docker-swarm-healthchecks]]
```

---

## 🔍 Workflow: Finding Info Quickly

### Scenario 1: "How to do X?"

```
1. Ctrl+O → Quick switcher
2. Type keyword: "health"
3. Choose: docker-compose-healthchecks.md
4. Specific section
5. ✅ Found in 10 seconds
```

---

### Scenario 2: "Which project used X?"

```
1. Open concept [[traefik-integration]]
2. Scroll to bottom → Backlinks panel
3. See: project-glasck-deployment
4. Click → Project details
5. ✅ Context retrieved
```

---

### Scenario 3: "Command for Y?"

```
1. Ctrl+O → "swarm cheat"
2. [[cheatsheet-docker-swarm]]
3. Ctrl+F → "logs"
4. docker service logs myapp
5. ✅ Command copied
```

---

### Scenario 4: "Why error Z?"

```
1. Ctrl+Shift+F → "hang" (global search)
2. Result: docker-swarm-overlay-networks
3. Section: Pitfall - MTU fragmentation
4. Solution: MTU 1450
5. ✅ Bug fixed
```

---

## 🔗 Link System

### 1. Hierarchical Links (Dependencies)

```markdown
**Prerequisites**: [[docker-basics]], [[docker-images-layers]]
**Related to**: [[kubernetes-basics]], [[docker-swarm-basics]]
```

**Usage**: Graph view shows learning path

---

### 2. Conceptual Links (Logical Connections)

```markdown
## Docker Swarm uses

- [[docker-images-layers]] to deploy services
- [[docker-network-isolation]] to secure communications
- [[docker-volumes]] for data persistence
```

**Usage**: Understand complete architecture

---

### 3. Reference Links (Practice)

```markdown
See also:
- [[cheatsheet-docker-swarm]] for quick commands
- [[project-glasck-deployment]] for real example
```

**Usage**: Quick navigation to practical resources

---

## 📊 Daily Maintenance

### Morning (5 min)

```
1. Daily note (optional)
   ├─ Day's objective
   └─ Concepts to practice

2. Check reviews
   ├─ Search: next-review < today
   └─ Review 1-2 concepts
```

---

### During Work

```
1. Document in real-time
   ├─ Pitfalls encountered
   ├─ Solutions found
   └─ Time invested

2. Create links as you go
   ├─ Concept → Project
   ├─ Concept → Concept
   └─ Graph view builds itself
```

---

### End of Day (5 min)

```
1. Update notes
   ├─ Complete sections in progress
   └─ Update tags if status changed

2. Git commit (auto if plugin)
   ├─ Changes saved
   └─ Sync between machines
```

---

## 📅 Weekly Maintenance

### Friday Afternoon (30 min)

```
1. Review statuses
   ├─ Search: tag:#status/learning
   ├─ Which concepts mastered?
   └─ Update tags → status/mastered

2. Graph check
   ├─ Ctrl+G → Graph view
   ├─ Orphan notes?
   └─ Missing connections?

3. Update cheatsheets
   ├─ New commands learned?
   └─ Add to cheatsheets
```

---

## 🗓️ Monthly Maintenance

### Last Day of Month (1-2h)

```
1. Global review
   ├─ Dataview: status/learning vs mastered
   ├─ Technologies progress
   └─ Completed projects

2. Consolidation
   ├─ Similar notes to merge?
   ├─ Concepts too fragmented?
   └─ Cleanup obsolete notes

3. Graph cleanup
   ├─ Important orphan notes?
   ├─ Create MOC if necessary
   └─ Fix broken links

4. Metrics update
   ├─ Dashboard update
   ├─ Time invested review
   └─ Next month goals
```

---

## 📊 Map of Content (MOC)

If your vault exceeds 50+ notes, create MOCs (index by technology).

### Example: Docker MOC

**File**: `meta/moc-docker.md`

```markdown
# 📁 Docker - Map of Content

## Core Concepts
- [[docker-images-layers]] ⭐⭐⭐ (mastered)
- [[docker-containers-lifecycle]] ⭐⭐ (mastered)
- [[docker-network-isolation]] ⭐⭐⭐ (mastered)

## Orchestration
- [[docker-compose-basics]] ⭐⭐ (mastered)
- [[docker-swarm-overlay-networks]] ⭐⭐⭐⭐ (mastered)
- [[docker-swarm-deployment-strategies]] ⭐⭐⭐ (mastered)

## Projects Using Docker
- [[project-transcendence]] (monitoring focus)
- [[project-glasck-deployment]] (swarm focus)

## Cheatsheets
- [[cheatsheet-docker]]
- [[cheatsheet-docker-compose]]
- [[cheatsheet-docker-swarm]]

## Common Troubleshooting
- MTU fragmentation → [[docker-swarm-overlay-networks#pitfall-1]]
- OOM kills → [[docker-containers-lifecycle#pitfall-2]]
- Healthcheck failures → [[docker-compose-healthchecks#pitfalls]]
```

---

## 🎯 Spaced Repetition System

### Long-Term Retention

```yaml
# In concept metadata
created: 2025-12-23
next-review: 2026-01-23          # +1 month
retention-check-1: 2025-12-30    # +7 days
retention-check-2: 2026-01-23    # +30 days
retention-check-3: 2026-04-23    # +3 months
```

### Review Workflow

```
Day +7 (retention-check-1):
├─ Reread note without looking at code
├─ Can explain concept? ✅
├─ Can list use cases? ✅
└─ Update tag if needed

Day +30 (retention-check-2):
├─ Explain concept out loud (no notes!)
├─ List pitfalls from memory
└─ Search commands in cheatsheet

Day +90 (retention-check-3):
├─ Apply in new project
├─ Check for missing info
└─ Update concept with new learnings
```

---

## 🔍 Advanced Search

### Obsidian Search Syntax

```
# All Docker concepts
path:concepts/docker

# Non-mastered notes
tag:#status/learning

# Documented pitfalls
"Pitfall" path:concepts

# Projects with Swarm
tag:#swarm path:projects

# Notes created this week
created:7d

# Combinations
path:concepts tag:#docker tag:#status/mastered
```

---

## 📊 Personal Dashboard (Optional)

**File**: `meta/dashboard.md`

```markdown
# 📊 DevOps Knowledge Dashboard

## 📈 Statistics

### Concepts
- **Total**: 9
- **Mastered**: 6 ✅
- **Learning**: 2 🟡
- **To Learn**: 1 📝

### Technologies
- **Docker**: ⭐⭐⭐⭐ (4/5) - 35h invested
- **Traefik**: ⭐⭐⭐⭐ (4/5) - 20h invested
- **Kubernetes**: ⭐⭐ (2/5) - 8h invested

### Projects
- **Completed**: 2 (Transcendence, Glasck)
- **In Progress**: 0
- **Planned**: 1 (Multi-node Swarm)

---

## 🎯 This Week

### Learn
- [ ] Review [[docker-swarm-overlay-networks]]
- [ ] Finish [[kubernetes-basics]]

### Apply
- [ ] Project: Deploy multi-node Swarm cluster

### Planned Reviews
\`\`\`dataview
TABLE next-review
FROM "concepts"
WHERE next-review <= date(today) + dur(7 days)
SORT next-review ASC
\`\`\`

---

## 📚 In Progress

\`\`\`dataview
LIST
FROM "concepts"
WHERE contains(tags, "status/learning")
\`\`\`

---

**Last update**: {{date}}
```

---

## ✅ Checklist: Good Workflow

### Daily
- [ ] Document learnings in real-time
- [ ] Create links between concepts
- [ ] Update tags if status changed
- [ ] Git commit (auto or manual)

### Weekly
- [ ] Review status/learning → mastered
- [ ] Graph view check
- [ ] Update cheatsheets
- [ ] Plan next week

### Monthly
- [ ] Dataview metrics review
- [ ] Consolidate similar notes
- [ ] Graph cleanup (orphans)
- [ ] Dashboard update

---

## 🎓 Summary: The 3 Golden Rules

### 1. **1 Note = 1 Masterable Concept**
- Can you explain it in 5-10 min? ✅
- Is it reusable? ✅
- → Then it's a good atomic note

### 2. **Links > Hierarchy**
- Don't get lost in complex folder structure
- Use [[links]] for connections
- Graph view will show natural organization

### 3. **Simple > Complex**
- Start minimal
- Add complexity if needed (MOC, Dataview)
- Goal = find info in <30s

---

## 📖 Real Examples

### Well Atomized (Your Current Vault ✅)

```
docker/
├── docker-images-layers.md         # 1 concept = image optimization
├── docker-containers-lifecycle.md  # 1 concept = container states
├── docker-network-isolation.md     # 1 concept = network security
└── docker-swarm-overlay-networks.md # 1 concept = Swarm networking

Why? Each note = concept masterable separately,
reusable in different projects.
```

### Too Fragmented (Avoid ❌)

```
docker/
├── docker-image-layers.md
├── docker-image-caching.md      # Too separated!
├── docker-image-buildkit.md     # Same concept
└── docker-multi-stage-1.md      # Too fragmented
```

### Not Atomic Enough (Avoid ❌)

```
docker/
└── docker-complete-guide.md  # 10,000 lines! Unmanageable
```

---

**Time investment**: 10-20 min/day for optimal workflow

**ROI**: Knowledge capitalized for life, retrieval <30s

**Next**: Start with templates in [[obsidian-setup]]

---

**Last update**: 2025-12-23
