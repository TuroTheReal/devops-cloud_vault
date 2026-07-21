---
tags:
  - concept
  - terraform
  - drift
  - import
  - iac
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 3h
---

# Reconcile Manual Drift (cutover → align → decommission) - Terraform

**Prerequisites**: [[terraform-fundamentals]], [[terraform-ignore-changes-partial-ownership]]
**Related to**: [[safe-production-changes]]

> [!note] Stub — `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> Safe pattern for a risky prod change: **cut over by hand** (API/console) to control
> timing and rollback, THEN **reconcile Terraform** with reality (`import` / adjust until
> the `plan` is empty), THEN **decommission** the old path. Gap-free = the hostname is
> never unserved.

---

## 🤔 When to Use?

### ✅ Good for
- Cutover of a foundational/shared resource (apex DNS, origin, LB) where a direct `apply` is too risky.

### ❌ Bad for
- A trivial, non-shared change → a normal `apply` is enough.

---

## 📚 Key Concepts

### 1. Manual cutover first
Control of timing + immediate rollback. See [[safe-production-changes]].

### 2. Reconcile TF next
`terraform import` + adjust config until the `plan` is **empty**. Reality becomes the source of truth, the code catches up.
> TODO(you): your real import commands.

### 3. Decommission last
Remove the old path (old product/hosting) once the new one is stable and TF is aligned.

### 4. Lock during the manual window
**Fact**: lock the stacks during the manual work so no `apply` races the human.

---

## ⚠️ Pitfalls Experienced
> TODO(you).

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developer.hashicorp.com/terraform/cli/import

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
