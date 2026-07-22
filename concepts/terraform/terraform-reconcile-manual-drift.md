---
tags:
  - concept
  - terraform
  - drift
  - import
  - iac
  - status/learning
created: 2026-07-21
difficulty: 4
time-to-master: 8h
---

# Reconcile Manual Drift (cutover → align → decommission) - Terraform

**Prerequisites**: [[terraform-fundamentals]], [[terraform-ignore-changes-partial-ownership]]
**Related to**: [[safe-production-changes]]

---

## 🎯 TL;DR (30 seconds)

> Safe pattern for a risky prod change: **cut over by hand** (API/console) to control
> timing and rollback, THEN **reconcile Terraform by editing the code to match reality**,
> THEN **decommission** the old path. Gap-free = the hostname is never unserved.

---

## 🤔 When to Use?

### ✅ Good for

- **Cutover of a shared/foundational resource** (apex DNS, origin, LB) where a direct `apply` is too risky.

### ❌ Bad for

- **Trivial, non-shared change** → a normal `apply` is enough.

---

## 📚 Key Concepts

### 1. Manual cutover first

Control of timing + immediate rollback. See [[safe-production-changes]].

### 2. Reconcile TF next

Make the code match the reality you created by hand: **edit the TF config so it describes the manual change** (e.g. the new domain name / origin). Loop: `plan` → `apply`; if valid, **push**, then re-run `plan`/`apply` to confirm it still reports **no changes** (proof code and reality agree). `terraform import` is an option when the resource isn't in state yet, but the path I mainly used was code-edit → plan → apply → push → re-apply-and-verify-empty.

**My understanding**:
The reliable signal isn't "the import worked", it's "a second apply reports no changes". That empty plan is what proves the code and the live system actually agree; anything else means I'm still drifting.

**Mental model**: move the furniture by hand first, then redraw the floor plan to match — the drawing (code) chases reality.

### 3. Decommission last

Remove the old path (old product/hosting) once the new one is stable and TF is aligned.

### 4. Lock during the manual window

**Fact**: lock the stacks during the manual work so no `apply` races the human.

---

## 💻 Minimal Example

### Context

You changed a domain by hand (cutover); now make the code match, gap-free.

### Loop

```bash
# 1. edit the TF so it describes the manual change (new domain / origin)
# 2. check the plan matches only what you did by hand, then apply + push
terraform plan     # expect: only the manual change
terraform apply
git push
# 3. re-run to PROVE code == reality
terraform plan     # expect: No changes.
```

The empty second plan is the proof, not `terraform import`.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: trusting a single apply

**Symptom**:

```text
`apply` succeeds, but the next plan still shows changes — code and reality never converged.
```

**What went wrong**: reconciliation was called "done" after one apply, without re-running to confirm.

**Solution**: re-run `plan`; it must be empty. Lock the stacks during the manual window so no `apply` races you.

**Lesson**: reconciliation is proven by an empty second plan, nothing else.

### Pitfall 2: reaching for `import` when the resource is already in state

It just adds noise. Edit the code to match reality, then verify with an empty plan.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Reliable signal that code == reality?</summary>

**Answer**: A second `apply` that reports "No changes" — the empty plan, not the import.
</details>

<details>
<summary><strong>Q2:</strong> Is `terraform import` required?</summary>

**Answer**: No — mainly edit-code → plan → apply → push → re-plan-empty; import only when the resource isn't in state yet.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developer.hashicorp.com/terraform/cli/import

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
