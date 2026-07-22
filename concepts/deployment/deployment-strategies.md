---
tags:
  - concept
  - deployment
  - release
  - blue-green
  - canary
  - status/learning
created: 2026-07-22
difficulty: 2
time-to-master: 4h
---

# Deployment / Release Strategies

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[safe-production-changes]], [[terraform-reconcile-manual-drift]], [[2026-07-cloudflare-pages-to-workers-migration]]

---

## 🎯 TL;DR (30 seconds)

> How you move users from the old version to the new one. The axis that matters:
> **how much traffic is exposed at once** and **how fast you can undo it**.

**Analogy**: opening a new bridge. Big-bang = close the old one overnight and hope. Canary = let a few cars cross first, watch, then open all lanes.

---

## 🤔 When to Use?

### ✅ Good for

- **Risky change to a live service** where downtime or a bad version has real cost.

### ❌ Bad for

- **Trivial change on a low-risk surface** → a normal deploy is fine; strategy machinery is overhead.

---

## 📚 Key Concepts

### 1. Big-bang

**Facts**: switch everyone to the new version at once. Simple, but the blast radius is 100% and rollback is another full switch.

### 2. Rolling

**Facts**: replace instances gradually (N at a time). No full second environment needed; rollback is slower (roll back instance by instance).

### 3. Blue-green

**Facts**: run two full environments (blue = current, green = new). Cut traffic over at the router; rollback = flip back. Needs double capacity briefly, but rollback is instant.

### 4. Canary / progressive rollout

**Facts**: expose the new version to a **small slice** first (a low-traffic host, or a % of traffic), watch metrics + a monitor, then promote to full. Smallest blast radius, best signal before full exposure.

**My understanding**:
The real lever is **where I can undo cheaply**. Cutting at the routing layer (DNS record / route) makes blue-green and canary reversible in one step, which is why I picked a canary for the riskiest hostname in the migration: observable, reversible, no maintenance window.

**Mental model**: opening a new bridge — big-bang closes the old one overnight and hopes; canary lets a few cars cross, watches, then opens all lanes.

---

## 💻 In practice (migration)

- Cut at the **routing layer** (DNS record / Worker route), never by mutating the running app in place.
- **Canary** the highest-risk hostname: low-traffic/demo first → observe → prod → observe.
- Keep the **rollback pre-written and rehearsed** (see [[safe-production-changes]]).
- Reconcile IaC to match the manual cutover afterwards → [[terraform-reconcile-manual-drift]].

---

## 💻 Schema — exposure over time

```text
Big-bang:    0% ───────────────────────► 100%           (one switch, blast radius = all)
Blue-green:  100% blue ──flip──► 100% green              (rollback = flip back, instant)
Canary:      5% ─► 25% ─► 100%  (observe at each step)   (rollback = route traffic back)
```

Cut at the **routing layer** (DNS record / Worker route) so every step is reversible.

---

## ⚠️ Pitfalls Experienced

- **No pre-written rollback**: a strategy is only "safe" if the undo is one ready command, not improvised.
- **Canary without a monitor**: exposing a slice is pointless if you're not watching the right signal during the window.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Real lever in choosing a strategy?</summary>

**Answer**: Where you can undo cheaply — cutting at the routing layer makes blue-green/canary reversible in one step.
</details>

<details>
<summary><strong>Q2:</strong> Why canary the riskiest hostname?</summary>

**Answer**: Smallest blast radius + best signal: observable, reversible, no maintenance window.
</details>

---

## 📊 Stats

```yaml
Total time: ~4h (learning) + applied in the migration
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: general release-engineering practice; [[safe-production-changes]].

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
