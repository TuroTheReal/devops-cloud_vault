---
tags:
  - guide
  - meta/practice
  - topic/change-management
  - topic/incident
aliases: [prod-change-discipline, rollback-first, change-blast-radius]
updated: 2026-07-22
---

# Safe production changes — discipline

How to make a risky change to live infrastructure without turning it into an incident. Learned the hard way.

## ✅ Before you touch prod (checklist)

- [ ] Rollback **written, state captured, copy-paste ready, rehearsed on a sandbox**
- [ ] Blast radius mapped against the **live system**, not just the IaC
- [ ] A **monitor** ready (poll host+path, show HTTP code + who serves; `000`/DOWN = the gap)
- [ ] For the riskiest change: a **canary** path (low-traffic first, or a fraction of traffic)
- [ ] The mutation runs via a **self-documenting script** (readable during a screen-share)

Then: monitor on → (canary →) cutover → observe → reconcile. Rollback stays one command away throughout.

---

## 🔙 Rollback-first

Never run a forward mutation on prod until the undo path is written (a script, not "I'll figure it out"), has the IDs/state it needs captured beforehand, is copy-paste ready, and ideally rehearsed on a throwaway env. The prepared rollback is what makes a risky change acceptable: when the change surprises you (it will), the revert is one command, not improvised under pressure. Rehearsing swap+rollback first also proves the rollback actually works.

## 🐤 Canary / progressive rollout

Don't big-bang the riskiest hostname. Cut over a **small slice first** (a low-traffic or demo host, or a fraction of traffic), watch the monitor + metrics, then promote to full. Because the cut happens at the **routing layer** (DNS record / Worker route), each step is reversible instantly, so the scariest migration runs observable and without a maintenance window. See [[terraform-reconcile-manual-drift]] for aligning IaC after the manual cutover.

## 📜 Self-documenting scripts (readable audit trail)

During an incident you (or a colleague watching your screen) must read, top-to-bottom, EXACTLY what was done, not a wall of `curl`/JSON. A prod script should:
- have a header comment: what it does, why, and how to roll back,
- echo each action with a timestamp + target + before/after (not fire commands silently),
- save the mutated resource's prior state to a file for audit and rollback,
- avoid cryptic dense one-liners.

"Illegible during an incident" is a real defect, not a style nitpick.

## 💥 Blast radius vs the LIVE system

The most dangerous surprises come from dependencies the code doesn't model. Drift is real: dashboard-managed records, aliases, or CNAMEs can flatten onto the very resource you're changing, and `terraform plan` won't show them. Before changing a shared/foundational resource (apex/root DNS, an origin, a shared LB, a base image):
- enumerate what points at it in the **live** system, not only the repo,
- look for indirect references: aliases, flattened CNAMEs, records with no rule of their own that fall through to its origin,
- assume "everything I didn't map" is bigger than "the one path I'm fixing".

A change that looks surgical in the IaC can have a wide real blast radius. → real example: [[cloudflare-worker-shadows-origin-rule]].

---

## 🎯 TL;DR

Rollback first, ready and rehearsed. Canary the riskiest cut. Scripts readable enough to survive a screen-share. Blast radius measured against the live system, because the code doesn't model the drift.
