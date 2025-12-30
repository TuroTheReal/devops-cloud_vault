# Templates Usage Guide - Quick Reference

## 🎯 TL;DR

**Target writing time**:
- Concept: **30-40 min**
- Project: **45-60 min**
- Cheatsheet: **20-30 min**

---

## 📝 Which Template to Use?

### For a CONCEPT

Always use: [meta/templates/concept-template.md](../templates/concept-template.md)

**When to use**:
- After a practical lab
- Extraction from a project
- Concept learned and tested

**Essential sections (YOUR WORDS)**:
1. ✅ **TL;DR + Analogy** (5 min) - Ultra-concise reformulation
2. ✅ **Key Concepts - My understanding** (15-20 min) - 3-5 concepts max
3. ✅ **Minimal Example** (10 min) - Tested code
4. ✅ **Pitfalls Experienced** (10 min) - Only if experienced

**Total time**: 30-40 min

---

### For a PROJECT

Always use: [meta/templates/project-template.md](../templates/project-template.md)

**When to use**:
- At the end of a project/lab
- Post-mortem extraction

**Essential sections**:
1. ✅ **What I Learned** (15 min) - 3-5 concepts max
2. ✅ **Challenges & Solutions** (20 min) - Pitfalls experienced
3. ✅ **Time Breakdown** (5 min) - 70/30 tracking

**Total time**: 45-60 min

---

### For a CHEATSHEET

Always use: [meta/templates/cheatsheet-template.md](../templates/cheatsheet-template.md)

**When to use**:
- Frequently used commands
- Extraction from projects

**Essential sections**:
1. ✅ **Quick Start** (5 min)
2. ✅ **Commands by Operation** (10 min)
3. ✅ **Troubleshooting** (5 min) - Errors experienced

**Total time**: 20-30 min

---

## 🚀 Optimized Workflow

### After a lab/project

**Example**: VPS Hetzner SSH hardening (2h30)

```
1. Fill project learnings (45 min)
   projects/2025-12-vps-hetzner/learnings.md

2. Extract concepts (3 × 15 min = 45 min)
   concepts/linux/linux-ssh-hardening.md
   concepts/linux/linux-firewall-ufw.md
   concepts/linux/linux-systemd-socket.md

3. Update cheatsheet optional (15 min)
   cheatsheets/linux/linux-security-admin-commands.md

Total documentation: 1h30 ✅
```

---

## ✅ Quick Checklist

### Concept complete?

- [ ] TL;DR + Analogy (simple and clear)
- [ ] When to Use (2 good + 1 bad)
- [ ] Key Concepts (3-5 max, YOUR WORDS)
- [ ] Minimal Example (tested code)
- [ ] Pitfalls (only if experienced)
- [ ] Stats (time + status)
- [ ] Obsidian Links (prerequisites, related)

**Time**: 30-40 min

---

### Project complete?

- [ ] Metadata (tags, duration, status)
- [ ] Context (objective, architecture, stack)
- [ ] What I Learned (3-5 concepts)
- [ ] Challenges & Solutions (2-4 max)
- [ ] Time Breakdown (70/30 tracking)
- [ ] Extractions TODO (concept links)

**Time**: 45-60 min

---

### Cheatsheet complete?

- [ ] Quick Start (install + basic usage)
- [ ] Commands by Operation (CRUD)
- [ ] Debug / Inspection
- [ ] Troubleshooting (errors experienced)
- [ ] Personal Notes (daily workflow)

**Time**: 20-30 min

---

## 💡 Tips

### Speed up writing

1. **Capture hot**: Document DURING the lab, not after
2. **Draft OK**: Better quick note than no note
3. **Max 3-5 concepts**: Focus on essentials
4. **Pitfalls = Gold**: Prioritize errors experienced
5. **Obsidian Links**: Always link to related concepts/projects

### Documentation workflow

```
During lab:
→ Note errors/solutions in draft file

After lab:
→ 45-60 min fill project-template
→ 3 × 30 min extract concepts
→ Total: ~2h doc max
```

### Obsidian Links

**Format**: `[[file-name]]` or `[[folder/file-name]]`

**Examples**:
- `[[docker-containers-lifecycle]]`
- `[[concepts/docker/docker-networking]]`
- `[[cheatsheets/docker/docker-commands]]`
- `[[projects/2025-12-vps-hetzner-init-setup/learnings]]`

**Benefits**:
- Graph view visualizes connections
- Backlinks show where concept is used
- Quick navigation Ctrl+Click

---

## 🎓 Philosophy

### Accept imperfection

```diff
- ❌ Perfect note never written
+ ✅ Usable draft captured hot

- ❌ All sections filled
+ ✅ Essential sections (YOUR WORDS)

- ❌ Perfectionist paralysis
+ ✅ Progressive documentation
```

### Focus on memorization

**High-value sections** (your words):
1. TL;DR + Analogy
2. Key Concepts - My understanding
3. Minimal Example
4. Pitfalls Experienced

**Low-value sections** (removed):
- Essential Commands (→ cheatsheet)
- Tests Done (rigid)
- External Resources (rarely used)
- Detailed Timeline (time-consuming)

---

**Last update**: 2025-12-30
